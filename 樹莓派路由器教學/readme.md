# 第一步:
設置靜態ip

![01.png](https://github.com/s84175/raspberrypi/blob/master/%E6%A8%B9%E8%8E%93%E6%B4%BE%E8%B7%AF%E7%94%B1%E5%99%A8%E6%95%99%E5%AD%B8/photo/01.png)
![02.png](https://github.com/s84175/raspberrypi/blob/master/%E6%A8%B9%E8%8E%93%E6%B4%BE%E8%B7%AF%E7%94%B1%E5%99%A8%E6%95%99%E5%AD%B8/photo/02.png)

#在鼠標位置案右鍵點擊”Wireless&Wire Network Settings”

#使樹梅派可以上網

# 第二步:
#安裝dnsmasq hostapd

sudo apt-get install dnsmasq hostapd
# 第三步:
#將無線接口wlan0配置成靜態地址

![03.png](https://github.com/s84175/raspberrypi/blob/master/%E6%A8%B9%E8%8E%93%E6%B4%BE%E8%B7%AF%E7%94%B1%E5%99%A8%E6%95%99%E5%AD%B8/photo/03.png)
# 第四步:
sudo nano /etc/network/interfaces#編寫interface

allow-hotplug wlan0 #wlan0可以熱插拔

iface wlan0 inet manual

source-directory /etc/network/interfaces.d

#每個版本的樹梅派interfaces這一文件的配置會不太一樣

#在interfaces這一文件加入這三行其餘的刪除
# 第五步:
sudo reboot

#dhcpcd restart後會分配一個靜態IP給無線接口wlan0

#reboot後生效
# 第六步:
#配置hostapd.conf

sudo nano /etc/hostapd/hostapd.conf

#內容如下:

interface=wlan0#這是上面配置過的 Wi-Fi 介面名稱

driver=nl80211#使用 nl80211 驅動當作 brcmfmac 驅動

ssid=pipi#這是你的 Wi-Fi 名稱

hw_mode=g #使用 2.4GHz 頻段

channel=6#使用頻道 6

wmm_enabled=1#啟動 WMM

macaddr_acl=0#接受所有 MAC addresses

auth_algs=1#使用 WPA 授權

ignore_broadcast_ssid=0#要求用戶端必須知道網路名稱

wpa=2#使用 WPA2

wpa_passphrase=12345678#網路密碼

wpa_key_mgmt=WPA-PSK#使用愈共享密鑰

rsn_pairwise=CCMP#使用 AES, 代替 TKIP
# 第七步:
#檢查配置是否成功

sudo /usr/sbin/hostapd /etc/hostapd/hostapd.conf

#如果顯示此圖內容即可進行下一步

![04.jpg](https://github.com/s84175/raspberrypi/blob/master/%E6%A8%B9%E8%8E%93%E6%B4%BE%E8%B7%AF%E7%94%B1%E5%99%A8%E6%95%99%E5%AD%B8/photo/04.jpg)

#退出後修改文件

sudo nano /etc/default/hostapd

#將#DAEMON_CONF=””修改為

DAEMON_CONF="/etc/hostapd/hostapd.conf"

#測試手機是否能夠連上

sudo /usr/sbin/hostapd /etc/hostapd/hostapd.conf
# 第八步:
#配置DNSMASQ

sudo nano /etc/dnsmasq.conf

#內容如下:

interface=wlan0#使用 wlan0 介面

bind-interfaces#綁定到介面，以確保我們不會發送其他地方的東西

server=218.2.2.2

server=114.114.114.114

server=8.8.8.8

domain-needed#不要轉發短域名

bogus-priv#不轉發未路由位址空間中的位址

dhcp-range=192.168.0.2,192.168.0.254,12h

#配發 IP 位址在 192.168.0.2 到 192.168.0.254 之間，租期為 12 小時
# 第九步:
#設置ipv4轉發

sudo nano /etc/sysctl.conf

#將此文件內的#字號去掉

#net.ipv4.ip_forward=1

#修改完後再命令列輸入

sudo sysctl –p

#使其立即生效(或者重啟完生效)
# 第十步:
#在有線接口eth0與無線接口wlan0建立NAT

#使有線網路分享到無線網路上

sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT

sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT

#將這些配置在開機時自動生效

sudo sh -c "iptables-save > /etc/iptables.ipv4.nat"

#編設rc.local

sudo nano /etc/rc.local

#在exit 0之前添加以下兩行

iptables-restore < /etc/iptables.ipv4.nat

sudo bash /etc/dnsmasq_delayinit.sh

#編設delayinit.sh,延時十秒使dnsmasq最後啟動以免發生錯誤

sudo nano /etc/dnsmasq_delayinit.sh

sleep 10

sudo service dnsmasq restart

sudo chmod +x /etc/dnsmasq_delayinit.sh
# 最後一步:
#重啟
sudo service dhcpcd restart

sudo service hostapd start

sudo service dnsmasq start

sudo reboot
