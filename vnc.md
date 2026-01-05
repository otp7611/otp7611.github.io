vnc

# 消息交互

```
VNCServerST::addSocket
 -> VNCSConnectionST::init
 -> SConnection::initialiseProtocol 发送第一条消息SConnection::RFBSTATE_PROTOCOL_VERSION
 
void mainloop(const char* vncserver, network::Socket* sock)
 -> CConn::CConn(const char* vncServerName, network::Socket* socket=NULL)
 -> CConnection::initialiseProtocol 准备接收CConnection::RFBSTATE_PROTOCOL_VERSION
```

## 时序图

### 版本协商

```
S(SConnection::RFBSTATE_PROTOCOL_VERSION) -> C(CConnection::RFBSTATE_PROTOCOL_VERSION) 服务端版本
C(CConnection::RFBSTATE_PROTOCOL_VERSION) -> S(SConnection::RFBSTATE_PROTOCOL_VERSION) 协商后版本
```

客户端收到服务器的版本消息后，会发送客户端协商后的版本信息给服务端。

客户端日志Using RFB protocol version可以查看协商后的版本。当前应该使用版本是3.8. 出现其它如3.3则认为是错误。

### 认证方式协商

```
S(SConnection::RFBSTATE_SECURITY_TYPE) -> C(CConnection::RFBSTATE_SECURITY_TYPES) 服务端所有支持的认证方式
C(CConnection::RFBSTATE_SECURITY_TYPES) -> S(SConnection::RFBSTATE_SECURITY_TYPE)  协商使用的认证方式
```

通过客户端日志Server offers security type可以看出服务端支持哪种认证。当前只使用一种认证方式secTypeVncAuth。日志Choosing security type可以看出最终使用哪个日志。

通过服务端日志Client requests security type可以看出客户端协商使用的认证方式。

### 认证初始化

```
SConnection::processSecurityTypeMsg 在选定认证方式后始终返回true
 -> SConnection::processMsg 返回true
 -> VNCSConnectionST::processMessages 因为SConnection::processMsg所以不退出loop,继续调用processMsg
  -> SConnection::processMsg
  -> SConnection::processSecurityMsg
  -> SecurityVncAuth::processMsg 如果没有发送Challenge数据，就发送Challenge数据。
```

```
S(SConnection::RFBSTATE_SECURITY) -> C(CConnection::RFBSTATE_SECURITY)  发送Challenge数据
C(CConnection::RFBSTATE_SECURITY) -> S(SConnection::RFBSTATE_SECURITY) 接收Challenge响应数据  这里请求用户输入pw
S(SConnection::RFBSTATE_QUERYING) -> C(CConnection::RFBSTATE_SECURITY_RESULT)  发送Challenge结果
```

服务端状态切换，

```
S(SConnection::RFBSTATE_QUERYING => SConnection::RFBSTATE_INITIALISATION) 通过虚函数void SConnection::queryConnection(const char* userName)让server去决定是否开始传输应用层，只要不出抛出异常就会进入RFBSTATE_INITIALISATION
```

客户端状态切换

```
C(CConnection::RFBSTATE_SECURITY_RESULT => CConnection::RFBSTATE_INITIALISATION)
```

如果初始化或SConnection::queryConnection异常，会导致客户端进入

```
CConnection::RFBSTATE_SECURITY_REASON 
```

客户端读取失败原因后退出。

如果正常，双方进入RFBSTATE_INITIALISATION。

### 应用层初始化

客户端在在认证初始化完成后会立即发送一个字节的share配置推动应用层初始化。

```
CConnection::processMsg
 -> CConnection::processSecurityResultMsg
 -> CConnection::securityCompleted
 -> writer_->writeClientInit(shared)
```

```
C(CConnection::RFBSTATE_INITIALISATION) -> S(SConnection::RFBSTATE_INITIALISATION) 发送是否share
S(SConnection::RFBSTATE_INITIALISATION) -> C(CConnection::RFBSTATE_INITIALISATION) 发送vnc参数
```

服务端

```
SConnection::processMsg
 -> SConnection::processInitMsg
 -> reader_->readClientInit()
 -> VNCSConnectionST::clientInit(bool shared)
 -> SConnection::clientInit(shared)
 -> 进入状态SConnection::RFBSTATE_NORMAL
```

客户端

```
CConnection::processMsg
 -> CConnection::processInitMsg
 -> reader_->readServerInit()
 -> CConnection::serverInit
 -> 进入状态CConnection::RFBSTATE_NORMAL
```



# 程序参数配置

```
class VncAuthPasswdParameter : public VncAuthPasswdGetter, BinaryParameter
class BinaryParameter : public VoidParameter
```

```
VoidParameter::VoidParameter(const char* name_, const char* desc_,
			     ConfigurationObject co)
  : immutable(false), name(name_), description(desc_)
{
  Configuration *conf = NULL;

  switch (co) {
  case ConfGlobal: conf = Configuration::global();
    break;
  case ConfServer: conf = Configuration::server();
    break;
  case ConfViewer: conf = Configuration::viewer();
    break;
  }

  _next = conf->head;
  conf->head = this;

  mutex = new os::Mutex();
}
```

VoidParameter把自己this放入对应的Configuration中。Configuration是一个静态对象。程序的任何地址都可以通过Configuration对象更新参数。

# 程序认证方式配置

以服务端为例

```
SConnection::processVersionMsg
 -> Security::GetEnabledSecTypes
```

把服务端所有支持的认证方式获取并传递给客户端。

```
SConnection成员SecurityServer security;
SecurityServer静态成员static StringParameter secTypes; 
SecurityServer(void) : Security(secTypes)

Security::Security(StringParameter &secTypes)
{
  char *secTypesStr;

  secTypesStr = secTypes.getData();
  enabledSecTypes = parseSecTypes(secTypesStr);

  delete [] secTypesStr;
}
```

secTypes可以通过Configuration配置。

这样就把Configuration中的配置转换到的enabledSecTypes。

# 像素格式转换PixelFormat::bufferFromRGB

如果源像素格式是RGB,目标像素是RGBA时,默认情况下,这个函数会把A通道设置为0. 在HTML中,如果A通道为0,则为透明的,所有如果希望显示出像素,就应该设置为0xff

```
        *r = *(src++);
        *g = *(src++);
        *b = *(src++);
        *x = 0xff;
```



