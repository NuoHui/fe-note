## 目录介绍

```shell

ls /  # 查看根目录
ls /root # 用户的家目录
ls /home/username # 普通用户的家目录
ls /etc # 配置文件目录
ls /bin # 命名目录
ls /sbin # 管理命令目录
ls /usr/bin  /usr/sbin # 系统预装的其他命令
```

## 帮助命令

man命令是manual的缩写。

```shell

man ls # 查看ls命令详情
man 空格 man # 查看man命令的第1章节内容
man 7 man # 查看man命名的第7章节内容
```

shell(命令解释器)自带的命名叫做内部命令，其他的是外部命令。

```shell

help cd # 内部命令我们使用help帮助

ls --help # 外部命令使用help帮助
```




我们使用type可以区分命名是属于外部命名还是内部命令。

```shell

type cd # cd is a shell builtin

```


