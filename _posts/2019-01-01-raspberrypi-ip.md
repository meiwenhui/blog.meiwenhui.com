### 启用SSH
在sd卡中新建一个名为ssh的文件即可

### 设置WIFI
https://www.raspberrypi.org/documentation/configuration/wireless/wireless-cli.md

编辑如下文件
/etc/wpa_supplicant/wpa_supplicant.conf
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=CN

network={
    ssid="WIFINAME"
    psk="wifipassword"
}
```

### 设置树莓派3 B+的静态IP
修改/etc/dhcpcd.conf 文件
```
sudo vim /etc/dhcpcd.conf
# 使用 vi 编辑文件，增加下列配置项

# 指定接口 eth0
interface eth0
# 指定静态IP，/24表示子网掩码为 255.255.255.0
static ip_address=192.168.1.20/24
# 路由器/网关IP地址
static routers=192.168.1.1
# 手动自定义DNS服务器
static domain_name_servers=114.114.114.114

interface wlan0

static ip_address=192.168.0.200/24
static routers=192.168.0.1
static domain_name_servers=192.168.0.1

```

修改完成后，按esc键后输入 :wq 保存。重启树莓派就生效了
``sudo reboot``

上面的配置文件中 , eth0是有线的配置  , wlan0是无线配置

ip_address就是静态IP , 后面要接/24

routers是网关

static domain_name_servers是DNS
