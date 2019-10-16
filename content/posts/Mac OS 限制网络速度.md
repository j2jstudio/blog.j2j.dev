# Mac OS 限制网络速度


## 使用Network Link Conditioner

通过Addtional Toos for Xcode中的Network Link Conditioner（网络链接调节器）实现网络速度控制。

### 安装步骤：

1. 安装Xcode
2. 安装Addtional Toos for Xcode ，下载地址[https://developer.apple.com/download/more/?=for%20Xcode](https://developer.apple.com/download/more/?=for%20Xcode), 或者通过菜单Xcode -> Open Developer Tool -> More Developer Tools 打开浏览器后搜索Addtional Toos for Xcode下载
![xcode-more-tools](./xcode-more-tools.png)
![More-Downloads-for Apple-Developers](./More-Downloads-for-Apple-Developers.jpg)

3. 打开Additional_Tools_for_Xcode -> Hardware -> Network Link Conditioner.prefPane 

![network-link-conditioner-dmg](./network-link-conditioner-dmg.png)
 
首次安装Network Link Conditioner 后，使用正常。当关闭并再次打开 	System Preferences，Network Link Conditioner配置面板将消失，尝试	重新安装时后出现错误: You can't install thee "Network Link 	Conditioner" preferences.  
	
 ![network-link-conditioner-installation-error](./network-link-conditioner-installation-error.png)
 
需要将用户PreferencePanes目录移动至系统配置目录。在命令行中输入如下命令：
	
```
sudo mv ~/Library/PreferencePanes/Network\ Link\ Conditioner.prefPane /Library/PreferencePanes/
```
	
再次打开System Preferences，Network Link Conditioner将正常展示。

### 使用

启用Network Link Conditioner会根据所选配置更改整个系统的网络环境，从而限制上行链路或下载带宽，延迟和数据包丢失率。

默认已经有了如下预设，可以直接选择使用：

* 100% Loss
* 3G
* DSL
* EDGE
* High Latency DNS
* LTE
* Very Bad Network
* WiFi
* WiFi 802.11ac

![network-link-conditioner-preset](./network-link-conditioner-preset.png)

也可以自行设定：

![network-link-conditioner-1m.png](./network-link-conditioner-1m.png)

启用后系统网络限制生效。


## 使用PF（OpenBSD's Packet Filter）工具限制

保存一下脚本pf-bandwidth-limit.sh

```

#!/bin/bash
set -o errexit    # always exit on error
set -o errtrace   # trap errors in functions as well
set -o pipefail   # don't ignore exit codes when piping output
set -o posix      # more strict failures in subshells
#set -x          # enable debugging


start_limit(){

  local size_kbytes="$1"
  # Reset dummynet to default config
  dnctl -f flush

  # Compose an addendum to the default config: creates a new anchor
  (cat /etc/pf.conf && echo 'dummynet-anchor "my_anchor"' && echo 'anchor "my_anchor"') | pfctl -q -f -

# Configure the new anchor
cat <<EOF | pfctl -q -a my_anchor -f -
no dummynet quick on lo0 all
dummynet out proto tcp from any to any port 1:65535 pipe 1
EOF

  # Create the dummynet queue
  dnctl pipe 1 config bw ${size_kbytes}Kbit/s

  # Activate PF
  pfctl -E

  # to check that dnctl is properly configured: sudo dnctl list

  sudo dnctl list

}

stop_limit(){
  sudo dnctl flush
  sudo pfctl -f /etc/pf.conf
  sudo dnctl list
}

main() {
  local size_kbytes=$((512)) # 512K

  case "$1" in
    start)
      start_limit "${size_kbytes}"  
    ;;
    stop)
      stop_limit
     ;;
    *)
      echo "Usage: $0 <start|stop>"
    ;;
  esac
}

main "$@"

```

```
sudo chmod a+x ./pf-bandwidth-limit.sh

#启动速率限制
sudo ./pf-bandwidth-limit.sh start
#停止速率限制
sudo ./pf-bandwidth-limit.sh stop
```
## 参考


* [https://nshipster.com/network-link-conditioner/](https://nshipster.com/network-link-conditioner/)
* [https://mop.koeln/blog/limiting-bandwidth-on-mac-os-x-yosemite/](https://mop.koeln/blog/limiting-bandwidth-on-mac-os-x-yosemite/)
* [https://thesmithfam.org/blog/2012/04/11/throttling-your-network-connection-on-mac-os-x/](https://thesmithfam.org/blog/2012/04/11/throttling-your-network-connection-on-mac-os-x/)
