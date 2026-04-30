ssh root@192.168.10.1

http://192.168.1.1/cgi-bin/luci/

5 ac/n
2.4 b/g/n

---

Настроить VPS 

Wireguard
79437110
Сервис аренды VDS/VPS RuVDS https://ruvds.com/ru-rub:
Логин: satovritti@gmail.com
Пароль: V582Qk3SY2

VPS:
ssh root@194.87.236.194
68mVQ64mzs

apt update
apt upgrade -y

# Amneziawg
Установить на нем amneziawg

Импортировать файл подключения для роутера, важно выбрать connection format - amneziawg native format

Настройка роутера
- Описание настройки для версии 23.05.3

- Network → Interfaces page, click on edit lan interface,
- Set LAN as static IPv4 address as 192.168.x.1 (with x different from the network to which you will connect via Wi-Fi),
-  Network → Wi-Fi, click on scan and choose the “network” link and click “Join Network”.
- Enter the Wi-Fi password, leave the “name of new network” as “WWAN” and select WWAN (or WAN) firewall zone. Click Save,
-  Network → Interfaces page, click on edit wwan interface,
- Move to the Firewall tab. Click on Save and Apply.
- Network → Firewall, click edit in wan zone and check WAN and WWAN in “covered networks”, click save and apply,

После подключения ВПН перезагрузить роутер.

Настройка amneziawg https://redshieldvpn.org/ru/help/vpn_on_routers/openwrt_routers/amneziawg2_openwrt

---
```
[Interface]

Address = 10.8.1.7/32

DNS = 1.1.1.1, 1.0.0.1

PrivateKey = kQv7bFxny/WtqrsK20rexx9JtyCr/MsPnxFMD740bF8=

Jc = 6

Jmin = 10

Jmax = 50

S1 = 55

S2 = 114

H1 = 1537812578

H2 = 622502062

H3 = 1643525873

H4 = 1021774241

  

[Peer]

PublicKey = WUITzbQKuhbVeFNDWobOYviTo0w8N1JbwppfxBmZAQ4=

PresharedKey = ijVltG+oglU0kMn3E7tbbFKhKynZMjZvnzQZ9OR8WiM=

AllowedIPs = 0.0.0.0/0, ::/0

Endpoint = 194.87.236.194:39368

PersistentKeepalive = 25
```

# OpenVPN

Список серверов https://vpnobratno.info/

Установить 
- openvpn-openssl and 
- luci-app-openvpn

Релогин и появится вкладка VPN-OpenVPN
- OVPN configuration file upload
- Выбрать файл -> set name -> upload
- Set enabled
- Save&Apply

Network -> Firewall
- WAN -> Edit
- Advanced Settings -> Covered devices
- Ethernet adapter -> tun0
- Save
- Save&Apply

Проверка
- https://2ip.ru/

---
ВПН наоборот

```
vpn://AAAAw3jaJYvdDoIgFIBfxXEdChItfRlHh2Oe6YAB2qr17kHdfj9vZgJN6Gzw5DIbG7bkHNLYdQ-88eLaAwO6FjaClZ2aX77is5YI4tIL3XMpB83PRiC_SoN8tkorMRg0aq4LeDfTfTowJvKunLJAiwkihfwnzJm4v5a95iH67MFvFbPPFz9cMqw
```