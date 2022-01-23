---
layout: post
title:  "building and setup of bitcoin core and lightning node on ubuntu server in aws ec2"
---

---

setting up the user and filesystem
==================================
for a filesystem in created a ebs volume with 500GB magnetic storage (i know not ideal but cheaper). i assigned it to the
ec2 instance, formatted and mounted it into the home directory of newly created btc user. all commands are ran as `root`.
check disks with `lsblk`.
```
fsdisk /dev/xvdX
mkfs.ext4 /dev/xvdX
mkdir /home/btcnode
mount /dev/xvdX /home/btcnode
UUID=$(blkid | cut -d '"' -f 2 | tail -n 1)
echo "UUID=$UUID /home/btcnode ext4 defaults 0 2" >> /etc/fstab
```
create the user
```
useradd btcnode
passwd btcnode
chown btc:btc /home/btcnode
```

---

dependencies for bitcoin-core and c-lightning
=============================================
```
apt install -y autoconf automake build-essential libtool autotools-dev pkg-config bsdmainutils \
  python3 libevent-dev libboost-dev libboost-system-dev libboost-filesystem-dev libboost-test-dev \
  libgmp-dev libsqlite3-dev python3-mako net-tools zlib1g-dev libsodium-dev gettext
```

---

building bitcoin-core from the git repository
=============================================
```
git clone https://github.com/bitcoin/bitcoin/ /home/btcnode/bitcoin_source
cd /home/btcnode/bitcoin_source
./autogen.sh
./configure --without-gui --without-miniupnpc
make
make install
```

---

building c-lighting from the git repository
===========================================
```
echo "building c lightning..."
pip3 install --user mrkd mistune==0.8.4
git clone https://github.com/ElementsProject/lightning.git /home/btcnode/lightning_source
cd lightning_source
./configure
make
make install
```

running the nodes
=================
* running bitcoind and lightningd manually
```
bitcoind &
./lightningd/lightningd &
./cli/lightning-cli help"
```
* use initd scripts
```
TODO
```

---

install script
==============
i created a install script for ubuntu server including everything i wrote here.
i am also planning on extending this with more applikations like, lnbits, ride the lightning, and many more.
Github link: [install-bitcoin](https://github.com/dni/scripts/blob/main/server/install-bitcoin.sh)

---

links
=====

* [install-bitcoin](https://github.com/dni/scripts/blob/main/server/install-bitcoin.sh)
* [bitcoin core build-unix](https://github.com/bitcoin/bitcoin/blob/master/doc/build-unix.md)
* [c-lightning ubuntu build](https://github.com/ElementsProject/lightning/blob/master/doc/INSTALL.md#to-build-on-ubuntu)
* [ride the lightning](https://github.com/Ride-The-Lightning)
* [lnbits](https://github.com/lnbits)
