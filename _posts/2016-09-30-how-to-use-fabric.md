---
layout:     post
title:      "Fabric使用简介"
subtitle:   "Fabric安装、语法、例子等介绍"
date:       2016-09-30
author:     "vcoolwind"
bg-color:   "linear-gradient(rgba(47, 255, 0, 0) 60%, rgba(255, 49, 2, 0.34)), linear-gradient(70deg, rgba(53, 187, 20, 0.56) 32%, rgba(222, 100, 117, 0.58))"
tags:
    - Fabric
    - python
description:  "fabric 安装、语法、例子等介绍"    
---

# Fabric使用简介
Fabric是什么 -- Fabric是基于python的一个module，基于ssh对远端服务器进行连接和管理。可以用于批量的服务器管理，大大提高运维效率。
基于python语言的灵活性，可以方便的定制各种运维需求。
官方的介绍如下，地址在[这里](http://www.fabfile.org/)：
> Fabric is a Python (2.5-2.7) library and command-line tool for streamlining the use of SSH for application deployment or systems administration tasks.

* 目录
{:toc}

### 安装
使用pip，一句话完成Fabric的安装。当然，前提是已经安装好了pip及python。

```bash
pip install fabric
```
### 命令自动补全插件
[fabric-completion](https://github.com/kbakulin/fabric-completion)是一款非常好用的fab命令自动补全插件，推荐使用。使用方法比较简单，不再赘述。


### 先跑起来
随便在一个目录，建立fabfile.py，内容如下：
```py
#!/usr/bin/python
# coding=utf-8
# vim: ft=python

from fabric.api import run

def show_os_type():
    run('uname -a') 
```

执行`fab`命令(-H host;-u user ;-p passwd)
```
fab show_os_type -H 192.168.0.101 -u youruser -p yourpwd
--------------------------------------------------------------
[192.168.0.101] Executing task 'show_OS_type'
[192.168.0.101] run: uname -a
[192.168.0.101] out: Linux batch 2.6.32-5-amd64 #1 SMP Sun Sep 23 10:07:46 UTC 2012 x86_64 GNU/Linux
[192.168.0.101] out: 
 
Done.
Disconnecting from 192.168.0.101... done.
```

执行分析：fabric利用ssh链接到目标服务器，使用指定的用户名、密码登陆，执行`uname -a`后返回并中断链接。

### fabric探测机制
fab在当前目录探测fabfile.py文件或fabfile文件夹（包含__init__.py），执行对应的命令。
> Fabric is capable of loading Python modules (e.g. fabfile.py) or packages (e.g. a fabfile/ directory containing an __init__.py). By default, it looks for something named (to Python’s import machinery) fabfile - so either fabfile/ or fabfile.py.

当然，也可以自定义文件名，不如 yourfab.py, 执行时指定加载文件 `fab -f yourfab.py XXXX`。

### env介绍
env 定义fabric默认的各种行为和配置，也可以用于任务间的数据共享
- env.port 		:默认SSH远程链接的端口
- env.user 		: 默认SSH远程链接的用户名
- env.password 	：默认SSH远程链接的密码
- env.parallel 	:任务是否并行执行，默认为false。
- env.pool_size	:设置并行执行任务时并发的进程数。
- env.shell    	：默认`/bin/bash -l -c`,在使用 run 等命令时会使用到，作为 shell外壳程序。
- env.warn_only	:在 run / sudo / local 遇到错误时究竟是警告还是退出，默认退出（false）。
- 主机密码不同时
	* env.hosts = ['tomcat@192.168.244.128','tomcat@192.168.244.129']
	* env.passwords = {'tomcat@192.168.244.128:22':'111111','tomcat@192.168.244.129:22':'111111'}
- 主机密码相同时
	* env.hosts = ['tomcat@192.168.244.128','tomcat@192.168.244.129']
	* env.password='passwd'


	

### 角色（主机组）定义
fabric用于批量服务器管理，那么提前预定好角色就十分重要，fabric通过env.roledefs定义角色名和对应的主机列表。
定义举例：
```python
env.roledefs.update({
    'openapi_predeploy': [
        '192.168.0.106'
    ],
    'openapi_deploy_part1': [
        '192.168.0.104',
        '192.168.0.27',
        '192.168.0.29',
        '192.168.0.118',
   ], 
   'openapi_deploy_part2': [
       '192.168.0.51',
       '192.168.0.15',
       '192.168.0.30',
    ],  
    'nginx': [
        '192.168.0.5',
        '192.168.0.6',
    ],
})
```
通过在对应的方法使用roles装饰器标注使用对应的角色
```python
@task
@roles('nginx')
@with_settings(user='root')
def showosname():
    run('uname -a')    
```
### 装饰器
fabfile 中可以方便使用的装饰器，用于指定使用的资源，并覆盖全局设置。
- fabric.decorators.parallel(pool_size=None) ：强制被装饰的函数并行执行而非同步执行，覆盖全局`env.parallel`设置。
- fabric.decorators.serial(func): 强制顺序执行，优先级高于`env.parallel`。如果任务同时被 serial 和 parallel 装饰器装饰，parallel 的优先级更高。
- fabric.decorators.roles(*role_list)：定义（服务器）“角色”名，然后用于寻找对应的主机列表。
- fabric.decorators.runs_once(func)：阻止函数多次执行的装饰器。通过保存内部状态，使用该装饰器可以保证函数在每个 Python 解释器中只运行一次。`runs_once 无法和任务并行执行同时生效。`
- fabric.decorators.task(*args, **kwargs) : 将函数封装为新式任务的装饰器。
- fabric.decorators.with_settings(*arg_settings, **kw_settings) : 作用等价于 fabric.context_managers.settings,将整个函数封装起来，其效果类似于执行在 settings 上下文管理器中。
装饰器综合举例如下：

```python
@runs_once
def runonce():
    print('i can  run once!')

@task
def runsoncetest():
    execute(runonce)
    execute(runonce)
    execute(runonce)

@task
@parallel(pool_size=3)
@roles('openapi_deploy_part1')
@with_settings(warn_only=True)
def show_os_type():
    run('uname -a')
```

### 上下文管理器
fabric通过使用with进行上下文管理
```python
fabric.context_managers.cd(path)
fabric.context_managers.lcd(path)
fabric.context_managers.path(path, behavior='append')
fabric.context_managers.prefix(command)
fabric.context_managers.hide
fabric.context_managers.settings(*args, **kwargs)
fabric.context_managers.quiet()
fabric.context_managers.shell_env(**kw)
```
- fabric.context_managers.cd(path)：切换远程工作目录，支持嵌套
```python
with cd('/var/www'):
    run('ls') # cd /var/www && ls
    with cd('website1'):
        run('ls') # cd /var/www/website1 && ls
```
- fabric.context_managers.lcd(path):切换本地工作目录，同样支持嵌套。
- fabric.context_managers.path(path, behavior='append')：设置PATH环境变量，用于直接执行shell命令。behavior默认是`append` --> e.g. PATH=$PATH:<path>；`prepend` --> PATH=<path>:$PATH；`replace` -->PATH=<path>。
- fabric.context_managers.prefix(command):对任何包裹在sudo和run里面的命令加上&&。
```python
with prefix('workon myvenv'):
    run('./manage.py syncdb')
```
对应的shell脚本如下：
```bash
workon myvenv && ./manage.py syncdb
```
prefix支持嵌套使用:
```python
    with prefix('cd /var/www'):
        run('ls')
        with prefix('cd html'):
            run('ls')
```
上面的代码等同于:
```bash
        cd /var/www && ls
        cd /var/www && cd html && ls
```
- fabric.context_managers.hide(*args, **kwds) : 隐藏指定的输出,groups中包括:
	* Status  状态信息,通常用户使用键盘终端,或者是server断开了连接,这些status信息很重要
	* aborts  中止信息，类似状态信息,仅仅到fabric当作是library的时候需要关闭,即使这个被关闭了,aborts仍然发生
	* warnings 警告信息,
	* running 运行过程中的一些输出,比如命令正在运行过程中或者file传输过程中的一些输出
	* stdout 标准输出 本地或者远程与命令相关的标准输出
	* stderr 标准错误输出 本地或者远程与命令相关的错误输出
	* user 用于生产的一些信息(比如本地打印的一些信息)
```python
def test_hide():
    with hide('stdout','stderr'):
        run("ls -al")
```
上面的例子将不会有任何输出，因为标准输出和标准输入都隐藏了。
- fabric.context_managers.settings(*args, **kwargs) ：设置一些fabric内置变量，覆盖env变量。
```python
def test_with_settings():
    with settings(hide('everything'),warn_only=true):
        run('uname -a')
```
- fabric.context_managers.quiet() 等同于`settings(hide('everything'), warn_only=True)`。
- fabric.context_managers.shell_env(**kw) : 设置shell变量
```python
with shell_env(ZMQ_DIR='/home/user/local'):
    run('pip install pyzmq')
```
eq
```bash
$ export ZMQ_DIR='/home/user/local' && pip install pyzmq
```

### fabric 常用接口
fabric是对ssh的一个集成工具，对我们而言只需要使用相应的接口，来高效的完成工作 ： 本地或者远端执行命令， 分发文件，收集文件，还有一些权限相关的操作。 这些fabric都给我们提供了对应的接口。
如下所示：
```python
run (fabric.operations.run)
sudo (fabric.operations.sudo)
local (fabric.operations.local)
get (fabric.operations.get)
put (fabric.operations.put)
prompt (fabric.operations.prompt)
reboot (fabric.operations.reboot)
```
- run (fabric.operations.run) 远端执行
Fabric 中使用最多的就是 run 方法了。run是用来在一台或者多台远程主机上面执行shell 命令。
> * 方法的返回值是可以通过变量来进行捕获
> * 可以通过变量的.failed 和 .succeeded 来检查命令是否执行成功
> * 还有一个很赞的就是 run 方法中执行命令的时候，可以支持awk 很给力
```python
@task
@with_settings(user='wangyf',host_string='127.0.0.1',password='1qaz2wsx')
def testrun():
    ret = run('uname -a')
    print ret.return_code,ret.failed,ret.succeeded
    #print result.code,'failed=%s' %result.failed
    with settings(warn_only='yes'):
        ret = run('cd /root')
        print ret.return_code,ret.failed,ret.succeeded
```

- local (fabric.operations.local) 本地运行
```python
local('pwd')
local('set -m ; /etc/init.d/tomcat restart') 如果是脚本，要加set -m 支持后台执行并返回状态，否则会报错
```

- get (remote_path,local_path) 从远端服务器下载到本地,功能跟scp一样。可以从远程主机下载备份，或者日志文件等等。
	* 通过参数 remote_path 指定远程文件的路径
	* 通过参数 local_path 指定远程文件的路径
```python
@task
@with_settings(user='wangyf',host_string='127.0.0.1',password='1qaz2wsx')
def testget():
    get('~/testget','/tmp/')
```

- put (local_path,remote_path,mode) 发布到远端服务器,和get类似。
	* local_path - 本地路径
	* remote_path - 远程路径
	* mode - 文件属性
```python
@task
@with_settings(user='wangyf',host_string='127.0.0.1',password='1qaz2wsx')
def testput():
    put('~/testget','/tmp/upload/',mode=644)
```
- fabric.operations.prompt(text, key=None, default='', validate=None)  提示用户输入并返回对应内容
	* key不为空时，输入值会注入env中，可以在任务间传递。
	* validate 可以说回调函数或正则表达式。	
	
```python
@task
@with_settings(user='wangyf',host_string='127.0.0.1',password='1qaz2wsx')
def testprompt1():
    num=prompt('请输入序号（数字）',key='unum', default='0',validate=int)
    print env.unum
    print '你输入了[%s]' %num

@task
@with_settings(user='wangyf',host_string='127.0.0.1',password='1qaz2wsx')
def testprompt2():
    num=prompt('请输入序号（数字）',key='unum', default='0',validate='\d+')
    print env.unum
    print '你输入了[%s]' %num
```

### 使用rsync进行文件同步
get、put、project.upload_project使用sftp进行传输，project.rsync_project提供了使用rsync进行同步的能力。
rsync_project仅仅是rsync的封装，需要本地及远端都需要安装rsync。
主要参数如下：
> `remote_dir`：唯一必须的参数，定义远端服务器的目录。
>
> `local_dir` :本地目录。 当local_dir以‘\’结尾时，是`拉取`操作，从服务器端同步到本地。反之，当不以‘\’结尾时，则是把本地文件同步到远端。
>
> `exclude`   :等同于rsync的exclude。
>
> `delete`    : a boolean controlling whether rsync‘s --delete option is used. If True, instructs rsync to remove remote files that no longer exist locally. Defaults to False.
>
> `default_opts`: the default rsync options `-pthrvz`, override if desired

```python
from fabric.contrib import project, console

@task
@with_settings(user='wangyf',host_string='127.0.0.1',password='1qaz2wsx')
def syncfile_push():
    project.rsync_project(
        remote_dir='/tmp/remote/',
        local_dir='/tmp/local/testdir',
        default_opts='-rclvpth',
        delete=True
    )


@task
@with_settings(user='wangyf',host_string='127.0.0.1',password='1qaz2wsx')
def syncfile_pull():
    project.rsync_project(
        remote_dir='/tmp/remote/testdir',
        local_dir='/tmp/local/',
        default_opts='-rclvpth',
        delete=True
    )

```

### 支持颜色输出
```python
@task
def printWithColor():
    from fabric.colors import *
    print red('红色')
    print green('绿色')
    print blue('蓝色')
    print white('白色')
    print yellow('黄色')
    print cyan('蓝绿色')
    print magenta('品红色')
```

### 常用命令
- 判读远端是否存在目录或文件
```python
    from fabric.contrib.files import exists
    if exists('/opt/tomcat/logs/catalina.out'):
        print 'catalina.out exist'    
    else:
        print 'catalina.out not exist'
```

- 判断远程主机的文件中是否存在文本
```python
    from fabric.contrib.files import contains
    if contains('/opt/tomcat/catalina.out','username1'):
        print "contains text"
    else：
        print "no contains"
```

### 一个通用的logger
```python
@runs_once
def initLogger():
    logname='fab'+'.log'
    logFileName="logs/%s"%logname
    logger = logging.getLogger("fabric")
    formater = logging.Formatter("%(asctime)s %(name)s %(levelname)s - %(message)s","%Y-%m-%d %H:%M:%S")
    file_handler = logging.handlers.RotatingFileHandler(logFileName, maxBytes=10485760, backupCount=5)
    file_handler.setFormatter(formater)
    stream_handler = logging.StreamHandler(sys.stderr)
    logger.addHandler(file_handler)
    logger.addHandler(stream_handler)
    logger.setLevel(logging.INFO)
    env.logger = logger
    return logger

@task
def testlog1():
    execute(initLoggerWithRotate)
    env.logger.info(green('1:i am fine'))

@task
def testlog2():
    execute(initLoggerWithRotate)
    env.logger.info(green('2:i am fine'))

@task
def testlog():
    execute(testlog1)
    execute(testlog2)
```

 
参考：[这里](https://www.52os.net/articles/fabric-useful-functions.html)和[这里](https://ruiaylin.github.io/2014/11/24/fabric/)
 