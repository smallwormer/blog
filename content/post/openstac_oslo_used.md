---

title: OpenStack Oslo.conf 使用

date: 2018-12-31T17:12:17+08:00


---

## OpenStack Oslo 项目 

随着OpenStack项目越来越多，在每个项目中都有一套处理自己配置文件的代码，也越来越多的重复的工作需要来做，而且这些代码也总是项目之间来回的`copy-and-paste`, 为此，社区提出了Oslo项目。Oslo项目的宗旨是提供一些列的OpenStack项目的公共库。用于解析OpenStack中命令行（CLI）或配置文件(.conf)中的配置信息

###  Oslo.conf 使用###

-  配置文件：

​    

​        用来配置OpenStack各个服务的配置文件 .ini风格，通常以.conf结尾

- 组(option groups):

​    

​        划分以组为单位的配置的组值，通常例如 [keystone_auth]， 默认为 [DEFAULT]

- 配置项:

​    

​        具体的配置项的值，在命令行中注册后既可以使用，例如：host_ip = 127.0.0.1 等等

- 使用步骤

​    

1. 在需要使用的配置信息的模块中进行声明配置项，包括配置型的类型，名称，默认值 以及帮助信息等

2. 创建一个可以配置管理器的对象，这个对象可以来存储这些值，例如 opts = []

3. 将上面的对象注册进全局的CONF中，这样会将上面对象中的配置项的key,value进行注册，这样就可以调用了

4. 配置配置文件，例如 freezer.conf 通常的路径为 ~/.freezer/freezer.conf 或者 /etc/freezer.conf

​    或者 /etc/freezer/freezer.



## 使用举例

###  声明配置项 config.py



```python
from oslo_config import cfg

#创建对象CONF，用来充当调用的容器

    CONF = cfg.CONF
#组配置项(读个配置项可以组成一个组对象)

    _COMMON = [

        cfg.IPOpt('bind-host',

                  default='0.0.0.0',

                  dest='bind_host',

                  help='IP address to listen on. Default is 0.0.0.0'),

        cfg.PortOpt('bind-port',

                    default=9090,

                    dest='bind_port',

                    help='Port number to listen on. Default is 9090'

                    )

    ]
```



    ### 组配置项



```python
paste_deploy = [

        cfg.StrOpt('config_file', default='freezer-paste.ini',

                    help='Name of the paste configuration file that defines '

                    'the available pipelines.'),

    ]

###也可以进行单个的配置项例如

    test = cfg.ListOpt("test",

                        default=["test1", "test2"]

                        help="list")
```

​    



###  注册



    ```python
#注册单个配置项

        CONF.register_opt(test)

#注册多个配置项的配置组

        CONF.register_opts(_COMMON)

#将配置项加载进命令行中显示（可以用命令行进行直接配置等）

        CONF.register_cli_opts(_COMMON)

# register paste configuration 声明组 paste_deploy

        paste_grp = cfg.OptGroup('paste_deploy',
    
                                 'Paste Configuration')
    



  

### 注册



``` python
CONF.register_group(paste_grp)

#声明配置项在组中，这里的paste_deploy 是声明的paste_deploy

# 例如(paste_deploy),必须在注册组中的配置项前声明组

        CONF.register_opts(paste_deploy, group=paste_grp)

#查找配置文件，也可以 ~/.freezer/freezer-api.conf /etc/freezer-api.conf /etc/freezer/freezer-api.conf

       default_config_files = cfg.find_config_files('freezer', 'freezer-api')

# 调用CONF.cfg的 __Call__方法

        CONF(args=args,

            project='freezer-api',

            default_config_files=default_config_files,

            version=FREEZER_API_VERSION
           )
```







### 配置文件 freezer.conf

这里使用Freezer进行举例

 vim /etc/freezer/freezer.conf


```
#默认组

[DEFAULT] 

[cors]

[cors.subdomain]

#认证组

[keystone_authtoken]

#组下的配置项

admin_tenant_name = service

admin_password = 99cloud

admin_user = freezer

auth_port = 35357
auth_host = 172.16.214.81

auth_protocol = http

[oslo_middleware]

[oslo_policy]

[paste_deploy]

config_file = /etc/freezer/freezer-paste.ini

[storage]
hosts = http://172.16.214.81:9200
number_of_replicas = 0
index = freezer
db = elasticsearch

```

### 配置项

