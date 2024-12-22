# Conda常用命令

## 帮助命令
查看帮助文本
```
查看所有命令的帮助：
conda -h

#查看某个命令的帮助：
conda 命令名 -h

#如：
conda rename -h
```

## 与环境相关的命令

创建一个虚拟环境
```
conda create -n 环境名
conda create -n 环境名 python=3.8
```

列出所有虚拟环境
```
conda env list
#或者
conda info -e
```

进入某个虚拟环境
```
conda activate 虚拟环境名
```

退出某个虚拟环境
```
conda deactivate 虚拟环境名
```

删除某个虚拟环境
```
conda remove -n 虚拟环境名--all
```

重命名某个虚拟环境
```
conda rename -n 现在的名称  新的名称
```

## 与包相关的命令
安装某个包
```
#建议先进入相应的虚拟环境再进行安装，不建议全部安装在base
conda install 包名

#进入某个虚拟环境之后也可以使用pip安装包
pip install 包名
```

移除某个包
```
conda remove 包名
```

查看已经安装的包
```
conda list
```

将包更新至最新版本
```
conda update 包名
conda update -n 环境名 包名
```

## 虚拟环境的保存位置
简要信息
```
conda info
```

完整配置
```
conda config --show
```

修改conda配置文件，C:\Users\14134\.condarc，添加envs_dirs:
```
channels:
  - defaults
envs_dirs:
  - D:\Anaconda3\envs
```