- RPC-API 的存在是为了快速的响应进程服务之间的调用请求。
- PRC 调用的过程为： 

```
graph LR
　　A(api.py)-->B(rpcapi.py)    
　　B(rpcapi.py)-->C(manager.py) 
```


1. cinder\cinder\scheduler\rpcapi.py

```
class SchedulerAPI(rpc.RPCAPI):

    """省略代码"""

    def say_hello(self, ctxt):
        version = '3.0'
        cctxt = self.client.prepare(version=version)
        # cast 异步调用, call 同步调用
        # 通过cast方式的远程调用，请求发送后就直接返回了；通过call方式远程调用，需要等响应从服务器返回。
        cctxt.cast(ctxt, 'say_hello')

```

2. cinder\cinder\scheduler\manager.py

```
class _SchedulerV3Proxy(object):
    
    """省略代码"""
    
    def say_hello(self, context):
        """Demo function. test say hello."""
        LOG.debug('===========manager say_hello==============hello,wangyue========================')
        #通知ceilmeter
        rpc.get_notifier("volume", CONF.host).info(context, '======scheduler say hello to you, wangyue======', None)

```

3. 调用rpcapi

```
from cinder.scheduler import rpcapi

"""省略代码"""

    def say_hello(self, req):
        LOG.debug('=============say hello begin===================')
        context = req.environ['cinder.context']
        authorize(context, 'storages')
        rpc = rpcapi.SchedulerAPI()
        rpc.say_hello(context)
        return webob.Response(status_int=202)

```