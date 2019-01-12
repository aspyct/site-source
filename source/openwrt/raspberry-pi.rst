OpenWRT on Raspberry Pi 3B/3B+
==============================

.. NOTE::
    OpenWRT 18.06.1 is not working on RPi 3B+ due to a problem with the GPU
    (which is responsible for the boot process).
    Support for the RPi 3B+ is available in the `snapshot versions`_, and will
    probably land in stable at 18.06.2.

    Everything works fine on the RPi 3B (non plus).

Update rpi firmware
-------------------

Before you install OpenWRT on your raspi, it's a good idea to update the firmwares.
With a running install of raspbian, run the following commands:

::

    $ sudo apt-get update
    $ sudo apt-get upgrade
    $ sudo rpi-update


Get signing keys
----------------

Since OpenWRT will probably be used as a firewall, it is essential to verify the authenticity of your downloads.
To do that, you'll need the GPG public keys from OpenWRT.

Figure out which version of OpenWRT you will install, and then
`download the appropriate key <https://openwrt.org/docs/guide-user/security/signatures>`_.
Unfortunately, it seems you can't download the key with `curl(1)`_ or `wget(1)`_,
so you'll have to get that from your browser directly.

It's always a good idea to check the key fingerprint as obtained through another connection,
like your 3G network instead of your home wifi.

::

    $ gpg openwrt.asc
    gpg: WARNING: no command supplied.  Trying to guess what you mean ...
    pub   rsa4096 2018-05-16 [SC] [expires: 2020-05-15]
        6768C55E79B032D77A28DA5F0F20257417E1CE16
    uid           OpenWrt Release Builder (18.06 Signing Key) <openwrt-devel@lists.openwrt.org>

If all is good, you can import the key.

::

    $ gpg --import openwrt.asc
    gpg: key 0F20257417E1CE16: public key "OpenWrt Release Builder (18.06 Signing Key) <openwrt-devel@lists.openwrt.org>" imported
    gpg: Total number processed: 1
    gpg:               imported: 1


Download/verify image
---------------------

Download the factory firmware, sum and signature files.
Refer to the `device page`_ to find the latest version.

::

    $ wget https://downloads.openwrt.org/releases/18.06.1/targets/brcm2708/bcm2710/openwrt-18.06.1-brcm2708-bcm2710-rpi-3-ext4-factory.img.gz
    $ wget https://downloads.openwrt.org/releases/18.06.1/targets/brcm2708/bcm2710/sha256sums
    $ wget https://downloads.openwrt.org/releases/18.06.1/targets/brcm2708/bcm2710/sha256sums.asc

Verify the sha sum and gpg signature

::

    $ sha256sum --ignore-missing -c sha256sums
    openwrt-18.06.1-brcm2708-bcm2710-rpi-3-ext4-factory.img.gz: OK

    $ gpg --verify sha256sums.asc
    gpg: assuming signed data in 'sha256sums'
    gpg: Signature made Fri 17 Aug 2018 03:31:45 PM CEST
    gpg:                using RSA key 0F20257417E1CE16
    gpg: Good signature from "OpenWrt Release Builder (18.06 Signing Key) <openwrt-devel@lists.openwrt.org>" [unknown]
    gpg: WARNING: This key is not certified with a trusted signature!
    gpg:          There is no indication that the signature belongs to the owner.
    Primary key fingerprint: 6768 C55E 79B0 32D7 7A28  DA5F 0F20 2574 17E1 CE16


The GPG warning above indicates that the key is not trusted.
That is normal, since we only imported it.
However a quick check of the fingerprint won't hurt.

Make the bootable SD card
-------------------------

.. DANGER::
    Make sure to write to the correct device.
    Use ``dmesg | tail`` to figure out which device you just inserted.

::

    $ gzid -d $ gzip -d openwrt-18.06.1-brcm2708-bcm2710-rpi-3-ext4-factory.img.gz
    $ sudo dd if=openwrt-18.06.1-brcm2708-bcm2710-rpi-3-ext4-factory.img of=/dev/sdb bs=1M status=progress; sync

First login
-----------

The device will need a minute or two to boot. Wait until your computer gets an IP address.

.. WARNING::
    The device will act as a DHCP server once powered up.
    You might want to keep it off your existing network until it's configured.

SSH into the device and change the root password.

::

    $ ssh root@192.168.1.1
    The authenticity of host '192.168.1.1 (192.168.1.1)' can't be established.
    RSA key fingerprint is SHA256:yJruTSql8kyzbKkcC0Ai8EKage9Sb3910wsyPaOlrBk.
    Are you sure you want to continue connecting (yes/no)? yes
    Warning: Permanently added '192.168.1.1' (RSA) to the list of known hosts.

    ... motd

    root@OpenWrt:~# passwd
    Changing password for root
    New password:
    Retype password:
    passwd: password for root changed by root

To facilitate initial setup, you may choose to set OpenWRT to be a DHCP client instead of sever.

::

    # uci set network.lan.proto=dhcp
    # uci commit
    # /etc/init.d/network restart

External links
--------------

- `Device page <https://openwrt.org/toh/raspberry_pi_foundation/raspberry_pi>`_
- `OpenWRT first login <https://oldwiki.archive.openwrt.org/doc/howto/firstlogin>`_
- `snapshot versions <https://downloads.openwrt.org/snapshots/targets/>`_

.. _`curl(1)`: https://manpages.debian.org/stretch/curl/curl.1.en.html
.. _`wget(1)`: https://manpages.debian.org/stretch/wget/wget.1.en.html
