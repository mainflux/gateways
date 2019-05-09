# MFX-Mini Gateway
MFX-Mini GW is based on 8Devices' [Lima board](https://www.8devices.com/products/lima).

Info about Lima board can be found [here](https://www.8devices.com/wiki/lima).

## Cross-compiling
Quick info about cross-compiling can be found [here](https://www.8devices.com/wiki/lima:build).

## WiFi
Need to edit 2 files:
- `/etc/config/wireless`
- `/etc/config/network`

### `/etc/config/wireless`
This file is needed to define STA mode.

```
root@Lima:/# cat /etc/config/wireless 

config wifi-device 'radio0'
        option type 'mac80211'
        option channel '11'
        option hwmode '11g'
        option path 'platform/qca953x_wmac'
        option htmode 'HT20'
        option disabled '0'

config wifi-iface
        option device 'radio0'
        option network 'wwan'
        option mode 'sta'
        option encryption 'psk2'
        option ssid 'my_ssid'
        option key 'my_password'
```

### `/etc/config/network`
This file is needed to add WWAN network interface.

```
root@Lima:/# cat /etc/config/network 

config interface 'loopback'
        option ifname 'lo'
        option proto 'static'
        option ipaddr '127.0.0.1'
        option netmask '255.0.0.0'

config globals 'globals'
        option ula_prefix 'fd63:6fe1:4c42::/48'

config interface 'lan'
        option type 'bridge'
        option ifname 'eth0'
        option proto 'static'
        option ipaddr '192.168.1.1'
        option netmask '255.255.255.0'
        option ip6assign '60'

config interface 'wan'
        option ifname 'eth1'
        option proto 'dhcp'

config interface 'wan6'
        option ifname 'eth1'
        option proto 'dhcpv6'

config interface 'wwan'
        option proto 'dhcp'
```

### Avahi
```
opkg install avahi-daemon
```
This will install `avahi-dbus-daemon` (default one).

Or via feeds and compilation:
```
 ./scripts/feeds install avahi-dbus-daemon
 ```

### Mosquitto
```
./scripts/feeds install mosquitto-ssl
```

#### Upgrade
```
scp ./bin/targets/ar71xx/generic/openwrt-ar71xx-generic-lima-squashfs-sysupgrade.bin root@openwrt.local:/tmp
```

Then in Minicom:
```
vi /lib/upgrade/keep.d/base-files
```
and add `/etc/config/wireless`.

Then:
```
sysupgrade -v /tmp/openwrt-ar71xx-generic-lima-squashfs-sysupgrade.bin
```
