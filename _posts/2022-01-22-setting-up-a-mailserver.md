---
layout: post
title:  "setting up a mailserver on ubuntu server"
---

---

after many tries and failures, i finally did it
===============================================

i recently stumpled upon this very nice [emailwiz](https://github.com/LukeSmithxyz/emailwiz) script from LukeSmithxyz on github. Inspired by that, i gave setting up my own mailserver another try. It was surprisingly simple, it is running for weeks now and i had no troubles so far.

---

modification
============
i modified the script a little bit more included the certbot stuff and patched some stuff i found on the github issues pages.
Github link: [install-email](https://github.com/dni/scripts/blob/main/server/install-email.sh)
neue version

---

multiple mailboxes and domains
==============================
for this i had browse the web a little while and found this 2 useful resources [postfix virtual](http://www.postfix.org/VIRTUAL_README.html)
and [linuxbabe](https://www.linuxbabe.com/redhat/host-multiple-mail-domains-in-postfixadmin-centos-rhel).

1. you have to add a new unix user for each mailbox
```
 useradd -aG mail newaccount
 passwd newaccount
```

2. expand the ssl certificate, you need to have the A and CNAME dns record working for the SSL certificate, see 5.
```
certbot certonly --webroot --agree-tos --text --non-interactive --webroot-path /var/www/html -d mail.domain1.com,mail.domain2.org --expand
```

3. generate the dkim keys and edit `signingtable`, `keytable` and `trustedhosts`.
```
opendkim-genkey -b 2048 -d domain2.org -D /etc/postfix/dkim/ -s mail-domain2 -v
chown opendkim:opendkim mail-domain2.private
echo "mail-domain2._domainkey.domain2.org domain2.org:mail-domain2:/etc/postfix/dkim/mail-domain2.private" >> /etc/postfix/dkim/keytable
echo "*.domain2.org" >> /etc/postfix/dkim/trustedhosts
echo "*@domain2.org mail-domain2._domainkey.domain2.org" >> /etc/postfix/dkim/signingtable
```

4. edit the file `/etc/postfix/virtual`.
for first time using more than 1 accounts you also need to configure postfix `main.cf` and add a file `virtual`.
important! do not add you main domain `domain1.com` to your `virtual_alias_domains`.
```
postconf -e "virtual_alias_domains = domain2.org"
postconf -e "virtual_alias_maps = hash:/etc/postfix/virtual"
cat <<EOF
postmaster@domain1.com   postmaster
office@domain1.com       oldaccount
admin@domain1.com        oldaccount
postmaster@domain2.org   postmaster
info@domain2.org         newaccount
EOF > /etc/postfix/virtual
```
---

5. add all the required dns record. reverse dns is not needed multiple times it is fine if your main domain works.
note: replace `127.0.0.1` with your ip address.
```
A     domain2.org        "127.0.0.1"
A     mail.domain2.org   "127.0.0.1"
MX    domain2.org        "10 mail.domain2.org"
TXT   domain2.org        "v=spf1 mx a:mail.domain2.org -all"
TXT   _dmarc.domain2.org "v=DMARC1; p=reject; rua=mailto:dmarc@domain2.org; fo=1"
```
the domainkey TXT record you can `cat /etc/postfix/dkim/mail-domain2.txt`. i shortened the rsa key, but it should look something like that:
```
TXT mail-domain2._domainkey.domain2.org "v=DKIM1; h=sha256; k=rsa; \""
"p=MIIBIjANBgkqhkiG9WbcWfa2FgonIQH/HHEpK5nFF5yJ"
"X2McsrH0Bm4JvekHgUVQlpboNCCP2+m1UlFQML4wIDAQAB"
```
---

mail client - mutt
==================
i also use mutt for mail and i found this very nice [mutt-wizard](https://github.com/LukeSmithxyz/mutt-wizard) script
also from LukeSmithxyz, for setting up the addresses. it is very well documented.
you need to install `neomutt` for this to work properly.

---

links
=====

* [emailwiz](https://github.com/LukeSmithxyz/emailwiz)
* [install-email](https://github.com/dni/scripts/blob/main/server/install-email.sh)
* [postfix virtual](http://www.postfix.org/VIRTUAL_README.html)
* [linuxbabe](https://www.linuxbabe.com/redhat/host-multiple-mail-domains-in-postfixadmin-centos-rhel)
* [mutt-wizard](https://github.com/LukeSmithxyz/mutt-wizard)
