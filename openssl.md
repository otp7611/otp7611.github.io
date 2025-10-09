# openssl证书配置

背景,在msys2 clang64环境编译官网的openssl会出现证书验证失败的情况,原因是本地根证书未正确配置.

## openssl s_client工具

使用openssl代码库时,验证的过程与openssl s_client是一样的.如果代码使用时出现验证失败,可以使用用相同代码编译出来的openssl程序来验证一下.比如,验证gosct.cn:3443的证书.你先可以在浏览器中把正常验证的证书下载下来.用openssl作比较.

```
Builtin Object Token_DigiCert Global Root G2.crt
TrustAsia DV TLS RSA CA 2025.crt
gosct.cn.crt
```

gosct.cn:3443是三级证书.

```
depth=2 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert Global Root G2
depth=1 C = CN, O = "TrustAsia Technologies, Inc.", CN = TrustAsia DV TLS RSA CA 2025
depth=0 CN = gosct.cn                                                                       
```

```
openssl s_client -showcerts  -connect gosct.cn:3443
CONNECTED(00000003)                                                                         depth=2 C = US, O = DigiCert Inc, OU = www.digicert.com, CN = DigiCert Global Root G2
verify return:1                                                                             depth=1 C = CN, O = "TrustAsia Technologies, Inc.", CN = TrustAsia DV TLS RSA CA 2025
verify return:1                                                                             depth=0 CN = gosct.cn                                                                       verify return:1                                                                             
```

openssl默认使用的根证书目录是/etc/ssl/certs/而在msys clang64环境下,这个目录是不存在的.所以模拟一下失败.

```
openssl s_client -showcerts  -connect gosct.cn:3443 -CApath /tmp/checkcert/
CONNECTED(00000003)    
depth=1 C = CN, O = "TrustAsia Technologies, Inc.", CN = TrustAsia DV TLS RSA CA 2025
verify error:num=20:unable to get local issuer certificate                                   verify return:1                                                                             depth=0 CN = gosct.cn                                                                       verify return:1 
```

返回了失败unable to get local issuer certificate.因为/tmp/checkcert/没有根证书DigiCert Global Root G2

-CApath指定目录,这个目录下的所有以hash.0这样格式的证书都会被加载.-CAfile指定一个根证书文件路径.

我们对Builtin Object Token_DigiCert Global Root G2.crt这个证书作一下 hash

```
openssl x509 -hash -in Builtin\ Object\ Token_DigiCert\ Global\ Root\ G2.crt -noout
607986c7
ln -s Builtin\ Object\ Token_DigiCert\ Global\ Root\ G2.crt 607986c7.0
```

```
/tmp/checkcert/607986c7.0 -> 'Builtin Object Token_DigiCert Global Root G2.crt'
```

这样就验证成功的.

再看-CAfile情况.

```
openssl s_client -showcerts  -connect gosct.cn:3443 -CAfile Builtin\ Object\ Token_DigiCert\ Global\ Root\ G2.crt
```

这样也是成功的.

## X509_STORE_load_locations

```
int X509_STORE_load_locations(X509_STORE *store, const char *file, const char *dirs);
```

file对应-CAfile参数,dirs对应-CApath参数.

## ca bundle

```
mingw-w64-clang-x86_64-ca-certificates
```

在msys clang64中,并没有包括所有根证书的目录,但是有一个/clang64/etc/ssl/certs/ca-bundle.crt这个把所有根证书打包在一起的crt文件.

可以通过命令查看

```
openssl storeutl -noout -text cacert.pem | grep 'DigiCert Global Root G2'
        Issuer: C=US, O=DigiCert Inc, OU=www.digicert.com, CN=DigiCert Global Root G2
        Subject: C=US, O=DigiCert Inc, OU=www.digicert.com, CN=DigiCert Global Root G2
```

并不是只有根证书才能打包在一起,只要是ca证书就可能打包在一起,比如gosct.cn_nginx/gosct.cn_bundle.crt就有两个证书.

```
CN = TrustAsia DV TLS RSA CA 2025
CN = gosct.cn 
```

```
openssl storeutl -noout -text gosct.cn_nginx/gosct.cn_bundle.crt
...
Total found: 2
```

## openssl x509工具

查看证书

```
openssl x509  -in Builtin\ Object\ Token_DigiCert\ Global\ Root\ G2.crt -noout -text
```



## openssl verify工具

openssl verify验证证书时,是一级一级直接验证的.

```
openssl verify  -CAfile TrustAsia\ DV\ TLS\ RSA\ CA\ 2025.crt gosct.cn.crt 
gosct.cn.crt: OK

openssl verify -CAfile Builtin\ Object\ Token_DigiCert\ Global\ Root\ G2.crt  TrustAsia\ DV\ TLS\ RSA\ CA\ 2025.crt
TrustAsia DV TLS RSA CA 2025.crt: OK
```



