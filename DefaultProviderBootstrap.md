#对象构造
创建DefaultProviderBootstrap时，需要传入ProviderConfig
#服务发布
##checkParameters
检查传入参数的合法性：\
Class proxyClass = providerConfig.getProxyClass(); \
T ref = providerConfig.getRef(); \
检查注入的ref是否接口实现类

##将处理器注册到配置指定的每个Server
对于在ProviderConfig指定的每个ServerConfig，都会做如下事 
###重复发布控制
String key = config.getInterfaceId() + ":" + config.getUniqueId() + ":" + config.getProtocol(); \
在发布服务时，会控制同一协议、同一接口重复发布到某个Server上的次数,每当重复发布的次数超过了config.getRepeatedExportLimit(),就抛出异常。
###构造请求调用器
providerProxyInvoker = new ProviderProxyInvoker(providerConfig); \
providerProxyInvoker是服务端调用链入口，它内部持有：\
FilterChain: 在收到消费端请求后，请求会流经FilterChain的每个Filter，FilterChain的编排顺序如下： \
内置filter[0]->内置filter[1]->......->内置filter[m]->自定义filter[0]->自定义filter[1]->......->自定义filter[n]->ProviderInvoker 

其中ProviderInvoker是真实调用已发布服务的地方，它内部的逻辑是以反射的方式调用请求里携带的方法：\
SofaResponse sofaResponse = new SofaResponse(); \
Method method = request.getMethod(); \
Object result = method.invoke(request.getRef(),request.getMethodArgs()); \
sofaResponse.setResponse(result); \
最后返回sofaResponse。 \
响应经过FilterChain写回消费端: \
自定义filter[n]->自定义filter[n-1]->......->自定义filter[0]->内置filter[m]-> \
内置filter[m-1]->......内置filter[0]

###初始化注册中心
根据在config里配置的注册中心的配置列表，创建Registry\



            

