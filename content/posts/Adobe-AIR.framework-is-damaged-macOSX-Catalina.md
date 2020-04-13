---
title: "Adobe AIR.framework” is damaged and can’t be opened"
date: 2020-04-13T16:05:59+08:00
draft: false
tags:
  - 今天搜了啥
  - MacOS
---

MacOS 升级 Catalina 后,打开Adobe Air出现错误：**“Adobe AIR.framework” is damaged and can’t be opened.**


可以通过如下方式移除安全隔离属性



```
cd /Library/Frameworks
sudo xattr -r -d com.apple.quarantine ./Adobe\ AIR.framework
```

<!--more-->

检查隔离属性是否存在



```
ls -l@ ./Adobe\ AIR.framework/
```

MAC 软件提示已损坏，需要移到废纸篓一般有两种解决方案：

1. **解决方法一：除这个应用的安全隔离属性**：

```
sudo xattr -r -d com.apple.quarantine app路径
```

2. **解决方法二：允许任何来源的APP**：

    ```
    sudo spctl --master-disable
    ```

    也可以重新开启

    ```
    sudo spctl --master-enable
    ```