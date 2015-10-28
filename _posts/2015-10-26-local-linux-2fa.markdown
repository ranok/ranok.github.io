---
layout: post
title:  "Local Linux 2FA Setup"
date:   2015-10-26 11:36:04
categories: security
---
In order to reduce the risk of an attacker remotely logging into my system if my password gets compromised, I wanted to add two-factor authentication (2FA) to my Linux system. 
There are a number of methods for this such as Duo, Google Authenticator/Authy and Yubikey, however the majority of these are aimed for logging into a network-connected system
whereas I wanted to ensure I could access my laptop when it was off the network (or to login to connect to a new wifi network). For this purpose, Yubikey (nano) looked the best
when coupled with the YubiPAM module for support on Linux and OS X. After brief testing, this works as 2FA for the screensaver, sudo and login with my password and a quick tap to 
the nano in an unused USB port.

A lot of this was taken from [this slightly outdated guide](https://blog.rootshell.be/2009/03/27/yubikey-authentication-on-linux/) as well as reviewing the source to troubleshoot.

### Setup and Building
1. ```sudo apt-get install libpam0g-dev``` for the PAM headers needed to build YubiPAM
2. ```git clone https://github.com/firnsy/yubipam.git``` to get the latest version of the YubiPAM source
3. ```cd yubipam; autoreconf --install; ./configure; make -j3; sudo make install``` to build and install
4. ```sudo addgroup yubiauth```
5. ```sudo touch /etc/yubikey```
6. ```sudo chgrp yubiauth /etc/yubikey /sbin/yk_chkpwd```
7. ```sudo chmod g+rw /etc/yubikey```
8. ```sudo chmod g+s /sbin/yk_chkpwd```

### YubiKey Prep
The YubiKey AES key must be known to provision the system, I had a free key sitting around so I didn't mind nuking it with the [ykpersonalize tool](https://www.yubico.com/products/services-software/personalization-tools/):

1. Generate a fixed UID for the key (public) by converting a string (1 byte minimum, 16-bytes max) to modhex with [Yubico's tool](https://demo.yubico.com/modhex.php): "text -> "ifhgieif"
2. Wipe the Yubikey and set the fixed UID: ```ykpersonalize -1 -ofixed=ifhgieif```
3. Record the key (will start with h:) to use with the ykpasswd instruction: ```ykpasswd -a -u <USER> -k <AES KEY> -o <OTP (TAP YUBIKEY)>```
4. If all worked, you should be able to verify by running: ```ykvalidate -u sirnarwhal <OTP (TAP YUBIKEY)>```

### Tying to PAM
The last step is to add the following line to your /etc/pam.d/common-auth file before all other entries to enable Yubikey 2FA:

```auth sufficient pam_yubikey.so no_passcode```

Once you have verified this works for login, screensaver, sudo, etc. (you should tap the Yubikey first, then enter password) change ```sufficient``` to ```required```.