Gluon script to enable special treatment for single nodes based on their macs.
You can than do slightly corrections to single nodes via an update especialy if you dont have ssh access.

basicly you HAVE to have your own repo of gluon-treatment where you put your Files for every single node you want to change,
the example repo will do nothing than deleting itself.

put this into your modules file similar like this and add gluon-treatment to your site.mk.
```
GLUON_SITE_FEEDS='v14tov15helper ssidchanger banner treatment'
PACKAGES_TREATMENT_REPO=https://github.com/viisauksena/gluon-treatment.git
PACKAGES_TREATMENT_COMMIT=74b423c678b00000000000000002a7
PACKAGES_TREATMENT_BRANCH=master
```

you have to make a file for each node you want modify with the name to the MAC Adress of br-client in uppercase letters : the MAC of the node is the same as n meshviewer but uppercase and with :
you have to make file executeable ! (chmod a+x ...)
See the example file in files/lib/gluon/treatment/macs. 

The script would run at one time (cronjob) and then check for treatment files and then delete them and the cronjob and the script itself, so nothing is left at the end.
And all Nodes are thereby "normalized" as intendet.

its a hack, but quiet convenient to correct things like deprecated branches. It has some drawbacks for sure, we dont care.

here are some example treatment contents, you can add as much and creativly as you want

```
#!/bin/sh
# set hostname
uci set system.@system[0].hostname=blablabla
uci commit

#!/bin/sh
# change fastd mtu (dangerous if wrong mtu, because loosing connection to Freifunk at all is possible)
sed -i s/1426/1280/g /etc/config/fastd
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
# force branch switch, ice if you want to remove old branches completly
uci set autoupdater.settings.branch=newbranch
uci commit

#!/bin/sh
# install a package later on this node
# (your opkg and module packages have to setup properly)
opkg update
opkg install gluon-status-page

#!/bin/sh
# change ssid of disrespectfull / harmfull node owners
sed -i s/"<ssid>"/"dieser Router ist geklaut"/g /var/run/hostapd-phy0.conf
killall -HUP hostapd

#!/bin/sh
... whatever you love to do ...
```

CC-BY
