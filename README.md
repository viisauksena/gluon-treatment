Gluon script to enable special treatment for single nodes based on their MACs.
You can than do slight corrections to single nodes via an update which is especially useful if you don't have ssh access.

Basically you HAVE to have your own repository of gluon-treatment where you put your files for every single node you want to change, the example repo will do nothing than deleting itself.

put it into your modules file - changing example to your repository - and add gluon-treatment to your site.mk.
```
GLUON_SITE_FEEDS='v14tov15helper ssidchanger banner treatment'
PACKAGES_TREATMENT_REPO=https://github.com/viisauksena/gluon-treatment.git
PACKAGES_TREATMENT_COMMIT=74b423c678b00000000000000002a7
PACKAGES_TREATMENT_BRANCH=master
```

You have to create a file for each node you want modify with the MAC address of br-client as its filename. The MAC of the node is the same as the primary MAC you can see in meshviewer.
See the example file in files/lib/gluon/treatment/macs.

The script will run once via a cronjob, check for treatment files, execute a matching one and delete everything afterwards.

It's a hack, but quite convenient to correct things like deprecated branches.

Here are some example treatment contents, you can add as much and creatively as you want.

```
#!/bin/sh
# set hostname
uci set system.@system[0].hostname=blablabla
uci commit

#!/bin/sh
# change fastd mtu (dangerous if wrong mtu, because loosing connection to Freifunk completely is possible)
sed -i s/mtu\ \'1426\'/mtu\ \'1280\'/g /etc/config/fastd
/etc/init.d/fastd restart

#!/bin/sh
# change geo or activate/deactivate
uci set gluon-node-info.@location[0].latitude=<LAT>
uci set gluon-node-info.@location[0].longitude=<LONG>
uci set gluon-node-info.@location[0].share_location=1
uci commit gluon-node-info

#!/bin/sh
# add ssh-key 
echo "ssh-rsa AAAAB3NzaC1yc2EA..SUPERSTRONGSSHKEY... mykey-name" >> /etc/dropbear/authorized_keys
echo "ssh-rsa AAAAB3NzaC1yc2EA..SUPERSTRONGSSHKEY2... otherkey-name" >> /etc/dropbear/authorized_keys

#!/bin/sh
# remove all ssh keys (deactivate ssh login)
rm /etc/dropbear/authorized_keys
/etc/init.d/dropbear restart

#!/bin/sh
# force branch switch, nice if you want to remove old branches completly
uci set autoupdater.settings.branch=newbranch
uci commit

#!/bin/sh
# install a package later on this node
# (your opkg and module packages have to be setup properly)
opkg update
opkg install gluon-status-page

#!/bin/sh
# change ssid of disrespectful / harmful node owners
sed -i s/"<ssid>"/"dieser Router ist geklaut"/g /var/run/hostapd-phy0.conf
killall -HUP hostapd

#!/bin/sh
... whatever you love to do ...
```

CC-BY
