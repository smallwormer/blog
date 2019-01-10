---
title: "Ansible_hostname"
date: 2019-01-10T10:44:51+08:00
---



## Ansible Hostname 模块修改

在特殊场景下，ansible无法识别 一些特殊的操作系统，使用hostname模块更改主机名。特别是一些自己更改过的操作系统。这里以Test OS 为例

错误信息：

hostname module cannot be used on platform Linux (Other linux)



解决办法:

Ansible  代码修改

1. 这里先找到ansible的hostname模块

   modules/system/hostname.py

```python
from ansible.module_utils.basic import (
    AnsibleModule,
    get_distribution,
    get_distribution_version,
    get_platform,
    load_platform_subclass,
)

class Hostname(object):
    """
    This is a generic Hostname manipulation class that is subclassed
    based on platform.

    A subclass may wish to set different strategy instance to self.strategy.

    All subclasses MUST define platform and distribution (which may be None).
    """

    platform = 'Generic'
    distribution = None
    strategy_class = UnimplementedStrategy

    def __new__(cls, *args, **kwargs):
        return load_platform_subclass(Hostname, args, kwargs)
```



这里new 构建方法 在init之前发生，会去判断是何种类型的操作系统，导入 了basic的模块 load_platform_subclass



2. 跟踪basic模块

   module_utils/basic.py

   ```python
   def load_platform_subclass(cls, *args, **kwargs):
       '''
       used by modules like User to have different implementations based on detected platform.  See User
       module for an example.
       '''
   
       this_platform = get_platform()
       distribution = get_distribution()
       subclass = None
   
       # get the most specific superclass for this platform
       if distribution is not None:
           for sc in get_all_subclasses(cls):
               if sc.distribution is not None and sc.distribution == distribution and sc.platform == this_platform:
                   subclass = sc
       if subclass is None:
           for sc in get_all_subclasses(cls):
               if sc.platform == this_platform and sc.distribution is None:
                   subclass = sc
       if subclass is None:
           subclass = cls
   
       return super(cls, subclass).__new__(subclass)
   ```

   这里对不同的操作系统以及版本会进行不同的使用方式

   ```python
   #获取是何种平台，一般为Linux 以及其他
   def get_platform():
       ''' what's the platform?  example: Linux is a platform. '''
       return platform.system()
   ```

   ```python
   # 获取其中的发行版本的具体distribution ，然后根据这种方式来进行操作
   def get_distribution():
       ''' return the distribution name '''
       if platform.system() == 'Linux':
           try:
               supported_dists = platform._supported_dists + ('arch', 'alpine', 'devuan')
               distribution = platform.linux_distribution(supported_dists=supported_dists)[0].capitalize()
               if not distribution and os.path.isfile('/etc/system-release'):
                   distribution = platform.linux_distribution(supported_dists=['system'])[0].capitalize()
                   if 'Amazon' in distribution:
                       distribution = 'Amazon'
                   else:
                       distribution = 'OtherLinux'
           except:
               # FIXME: MethodMissing, I assume?
               distribution = platform.dist()[0].capitalize()
       else:
           distribution = None
       return distribution
   ```



   ## 修复方法

   1. 首先我们需要知道我们无法使用的平台的发行版 是基于发行版修改的（相同发行版一般修改主机名的方式是一致）
   2. 如果不是基于其他的发行版本变更的，需要自己编写Hostname模块适应



   ```python
   # module_utils/basic.py
   def get_distribution():
       ''' return the distribution name '''
       if platform.system() == 'Linux':
           try:
               supported_dists = platform._supported_dists + ('arch', 'alpine', 'devuan')
               distribution = platform.linux_distribution(supported_dists=supported_dists)[0].capitalize()
               if not distribution and os.path.isfile('/etc/system-release'):
                   distribution = platform.linux_distribution(supported_dists=['system'])[0].capitalize()
                   if 'Amazon' in distribution:
                       distribution = 'Amazon'
                   # 已知了这种发行版是基于Ubuntu的
                   elif: 'Test' in distribution:
                       distribution = 'Ubuntu'
                   else:
                       distribution = 'OtherLinux'
           except:
               # FIXME: MethodMissing, I assume?
               distribution = platform.dist()[0].capitalize()
       else:
           distribution = None
       return distribution
   ```



   3. 增加对应的distribution 对应的hostname去匹配

      ```python
      #  modules/system/hostname.py
      # 增加一个class 使其按照 DebianStrategy的更改主机名方式去更改主机名
      class TestHostname(Hostname):
          platform = 'Linux'
          distribution = 'Ubuntu'
          strategy_class = DebianStrategy 
      ```

      如此 即可执行hostname模块去适配



      ### 如何写想要hostname模块想要的操作



      ```python
      #modules/system/hostname.py
      # 编写自己的 strategy，参考debian 以及redhat系列即可
      # example
      class DebianStrategy(GenericStrategy):
          """
          This is a Debian family Hostname manipulation strategy class - it edits
          the /etc/hostname file.
          """
      
          HOSTNAME_FILE = '/etc/hostname'
      
          def get_permanent_hostname(self):
              if not os.path.isfile(self.HOSTNAME_FILE):
                  try:
                      open(self.HOSTNAME_FILE, "a").write("")
                  except IOError as e:
                      self.module.fail_json(msg="failed to write file: %s" %
                                                to_native(e), exception=traceback.format_exc())
              try:
                  f = open(self.HOSTNAME_FILE)
                  try:
                      return f.read().strip()
                  finally:
                      f.close()
              except Exception as e:
                  self.module.fail_json(msg="failed to read hostname: %s" %
                                            to_native(e), exception=traceback.format_exc())
      
          def set_permanent_hostname(self, name):
              try:
                  f = open(self.HOSTNAME_FILE, 'w+')
                  try:
                      f.write("%s\n" % name)
                  finally:
                      f.close()
              except Exception as e:
                  self.module.fail_json(msg="failed to update hostname: %s" %
                                            to_native(e), exception=traceback.format_exc())
      ```


