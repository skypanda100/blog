---
layout: post
title: 增加swap大小
---
* 增加swap大小, 30G 左右
```Bash
$ dd if=/dev/zero of=/public/swap bs=1024 count=30720000
```

* 设置交换文件
```Bash
$ mkswap /public/swap
```

* 立即激活启用交换分区
```Bash
$ swapon /public/swap
```

* 添加系统引导时自启动运行
  1. 打开/etc/fstab
  ```Bash
  $ vi /etc/fstab
  ```

  2. 将光标移动至最后一行按o键（意思是在光标行的下面添加一行）
  ```Bash
  /public/swap               swap                    swap    defaults        0 0
  ```

  3. 退出保存，先按退出键，在敲`:wq`，最后回车


