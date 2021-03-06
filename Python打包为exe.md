# 使用Pyinstaller将Python项目打包为exe

使用 pyinstaller ：

```
pip install pyinstaller
```

首先，使用在项目根目录使用 

```
pyinstaller -c -i icon.ico main.py
```

会新建一个dist目录，通过cmd进入后直接运行 main.exe ，通过控制台可以检查打包是否成功，

```
ModuleNotFoundError: No module named 'neobolt.bolt._io'
```

大概率会出现类似以上这种缺失模块的错误，需要在main.py同文件夹中寻找main.spec文件，打开后，将缺失模块添加到

```
hiddenimports=['neobolt.bolt._io']
```

直到运行成功，之后使用命令：

```
pyinstaller -F -w -i icon.ico main.py
```

生成一个单独的exe文件



## 问题

- 配置文件问题

  如果有图标，配置文件等额外文件，需要手动放到与原main.py**相同的相对路径**下

- 使用apscheduler定时框架

  使用触发器可能会有如下错误出现：

```
LookupError: No trigger by the name "cron" was found
```

​		这是因为pyinstaller没有打包cronTrigger，需要手动引入触发器：

```
from apscheduler.triggers.cron import CronTrigger
...
cron = CronTrigger(**self.date)
sched.add_job(func,trigger=cron,id='xx')
...
```

- 打包exe文件过大的问题

  这是因为pyinstaller会将项目python环境下的库统一打包，使用环境管理器anaconda或者virenv创建纯净环境，之后安装所需要的库即可，下面是conda创建纯净环境的命令：

  ```
  conda create --no-default-packages --name env_name --prefix "$full_python_path" "pkg1" "pkg2" "pkg3"
  ```

  --prefix是指向anaconda安装路径，不填则使用默认目录

  “pkg1"是创建时需要加入的模块，也可不填

  在默认环境下可以大大减少打包后exe的体积

- 递归深度

  ```
RecursionError: maximum recursion depth exceeded in comparison
  ```
  
  可以将递归的深度修改的大一些，即可解决问题，但是还是建议在程序中不要使用太深的递归层数。

  ```
import sys
  sys.setrecursionlimit(100000) #例如这里设置为十万 
  ```

- ‘NoneType’问题

  在  ```pyinstaller -c xxx.py```  命令下调试只报 ’NoneType‘ 错，网上各种替换文件或者升级版本方法均无用，于是<u>写了个类似的简版代码test.py测试</u>，终于开始报其他错误（我遇到的都是各种ImportError），耐心在test.spec文件中的hiddenimport添加各种包，测试代码通过后，将hiddenimport项复制到原版xxx.spec中，重新使用 ```pyinstaller -F xxx.spec``` 生成，通过！

  tips:  pyinstaller 控制台模式大多时候可以暴露问题，但是依然可能会隐藏核心问题，所以在控制台模式下定位到代码出错处还不能解决问题后，可以试着将bug处核心提出，单独测试

- win或者Linux平台问题

  直接在对应平台安装pyinstaller即可，会根据平台生成对应可执行文件。