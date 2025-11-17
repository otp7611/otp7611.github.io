wasm | emscripten 的使用

# nginx配置

/etc/nginx/mime.types

```
application/wasm                      wasm;
```

location配置

```
    location /bazel-bin {
        root /path/to/file;
        add_header Access-Control-Allow-Origin *;
        add_header Cross-Origin-Opener-Policy same-origin;
        add_header Cross-Origin-Embedder-Policy require-corp;
    }
```

