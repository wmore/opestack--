- api获取service list:

```
 get http://172.24.2.225:8776/v2/1967629bf8564fa6b2dee0db99477d47/os-services?host=node2
{
  "services": [
    {
      "status": "enabled",
      "binary": "cinder-scheduler",
      "zone": "nova",
      "state": "up",
      "updated_at": "2017-05-25T03:42:05.000000",
      "host": "node2",
      "disabled_reason": null
    },
    {
      "status": "enabled",
      "binary": "cinder-backup",
      "zone": "nova",
      "state": "up",
      "updated_at": "2017-05-25T03:42:54.000000",
      "host": "node2",
      "disabled_reason": null
    }
}
```

- 代码逻辑：
1. cinder\api\contrib\services.py\ServiceController.index()
```
class ServiceController(wsgi.Controller):
    
    '''省略'''
    
    def index(self, req):
    
        '''省略'''
        
        # 获取当前UTC时间
        now = timeutils.utcnow(with_timezone=True)
    
        '''省略'''
        
        # 数据库查询cinder.services表
        services = objects.ServiceList.get_all(context, filters)

        svcs = []
        # 遍历service查询记录
        for svc in services:
            # 用当前时间减去（updated_at或者created_at）
            updated_at = svc.updated_at
        
            delta = now - (svc.updated_at or svc.created_at)
            delta_sec = delta.total_seconds()
            # 如果记录有modified_at
            if svc.modified_at:
                delta_mod = now - svc.modified_at
                # 如果当前时间减去modified_at时间差更小，用这个时间差替换delta
                if abs(delta_sec) >= abs(delta_mod.total_seconds()):
                    updated_at = svc.modified_at
            # 如果delta的值小于cind.conf里配置项service_down_time（默认60s），则设置服务状态为up
            alive = abs(delta_sec) <= CONF.service_down_time
            
            art = (alive and "up") or "down"

```

补充python中and-or语法 ：


> 使用 and 时，在布尔上下文中从左到右演算表达式的值，如果布尔上下文中的所有值都为真，那么 and 返回最后一个值。
> 如果布尔上下文中的某个值为假，则 and 返回第一个假值
>     
> 使用 or 时，在布尔上下文中从左到右演算值，就像 and 一样。如果有一个值为真，or 立刻返回该值
> 如果所有的值都为假，or 返回最后一个假值
> 注意 or 在布尔上下文中会一直进行表达式演算直到找到第一个真值，然后就会忽略剩余的比较值


