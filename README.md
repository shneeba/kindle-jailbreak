K4 index

Index of the K4 category threads, posts and off-site resources.
K4 index


[edit] Jailbreak

The jailbreak itself does not yet allow you to do much. In fact, it only installs an additional "developer" key on the device, allowing for the installation of additional packages via the Kindle's own update mechanism.

This method works on Kindle 4 version 4.0.0 through 4.1.4.

Carefully read all of the instructions below, and follow them only after you have read everything and are sure what to do.
Universal method
Kindle in Diagnostics Mode.

    Download and unzip the jailbreak. 

    Plug in the Kindle and copy the data.tar.gz & ENABLE_DIAGS files plus the diagnostic_logs folders to the Kindle's USB drive's root
    Safely remove the USB cable and restart the Kindle (Menu -> Settings -> Menu -> Restart)
    Once the device restarts into diagnostics mode, select "D) Exit, Reboot or Disable Diags" (using the 5-way keypad)
    Select "R) Reboot System" and "Q) To continue" (following on-screen instructions, when it tells you to use 'FW Left' to select an option, it means left on the 5-way keypad)
    Wait about 20 seconds: you should see the Jailbreak screen for a while, and the device should then restart normally
    After the Kindle restarts, you should see a new book titled "You are Jailbroken", if you see this, the jailbreak has been successful. 

[edit] What's next?

Well, check the Index for misc stuff (custom screen savers? fonts?), but for more fun stuff, you'll want to gain SSH access to this thing. You have two possibilities, one of them depending on the age of your device:
[edit] SSH

SSH in diagnostics mode is disabled on newer devices (black Kindles from fall 2012, and potentially some silver ones, too, depending on their assembly date) but you can install the USBNet package which, among other things, will setup an SSH server on your device.

To install the USBNet package download kindle-usbnetwork-0.xx.N-k4.zip from the list of attached files and follow the instructions in the included README_FIRST.txt.

(Windows and OS X users, see the relevant passage in the next section for the gory details on how to deal with the network configuration).
[edit] Useful Commands

The following commands can be run from an ssh session.

Refresh e-books (useful if you have copied e-books over via a non-USB method e.g scp):

dbus-send --system /default com.lab126.powerd.resuming int32:1

Disable / Enable screensaver (0 for enable, 1 for disable):

lipc-set-prop -- com.lab126.powerd preventScreenSaver <1|0>

Note: This has the added benefit of keeping WiFi alive indefinitely when disabled.
[edit] lipc

The lipc tools available on the kindle allow the setting of properties that modify behaviours of a number of kindle functions.

To get a list of all lipc properties:

lipc-probe -a

To get the specific value of a property:

lipc-get-prop -- com.lab126.<subsystem> <property>
e.g
lipc-get-prop -- com.lab126.powerd status

To set the specific value of a property:

lipc-set-prop -- com.lab126.<subsystem> <property> <value>
e.g
lipc-set-prop -- com.lab126.powerd preventScreenSaver 1

[edit] SSH (on diags, potentially deprecated)

If you have a silver Kindle (the one that came out in fall 2011), depending on its assembly date, there may be a slight chance the SSH server installed on the diags partition is still working. For most people, you'll instead want to use the full proper USBNetwork hack, as mentioned in the previous section of this wiki page.

    Jot down your Kindle S/N (from the Settings menu).
    Plug in the Kindle and create a blank file named ENABLE_DIAGS in the Kindle's USB drive's root
    Safely remove the USB cable and restart the Kindle (Menu -> Settings -> Menu -> Restart). It should reboot in diagnostics mode.
    Plug in the USB cable and go to usb networking: Misc individual diagnostics -> Utilities -> Enable USBnet -> Exit. This will enable network access to your Kindle via USB.
    Depending on your OS, you might need to install a device driver. This mostly concerns old-ish Windows systems, see this page for the gory details.
    Setup your new network interface: IP 192.168.15.201 & subnet 255.255.255.0 

For the sake of illustration, that means something along the line of ifconfig usb0 192.168.15.201 on Linux. On OS X & Windows, use their dedicated Network Manager UI, taking care of properly setting up the subnet too. And, yes, use those exact settings, the Kindle expects them. If you hit a snag, know that a few OS-specific corner-cases are discussed in the doc bundled in the USBNetwork hack.

    Connect to root@192.168.15.244 using your favorite SSH client (Mac OS users can use Terminal, type ssh root@192.168.15.244). If you unbricked your Kindle with an image from one of our debricking threads, chances are the password will be mario.
    If it isn't, you have a few choices: with your S/N in hand,
        use the online password calculator to find out your password,
        use the info command from KindleTool to get the password, or
        login as framework (this time the password *will* be mario), and export the /etc/shadow file to run it through a cracking tool like John the ripper. 
    And you're in :)
    See points 3 & 4 in the Jailbreak section for more details on how to make your Kindle reboot to the main, full system. 

[edit] Disabling updates

If you are paranoid, you can prevent any updates (including automatic ones via WLAN) by renaming `/etc/uks` to e.g. `/etc/uks.disabled` after you gain root access. This will cause any update to fail with error U007, because the update process can no longer access the necessary keys.

Don't forget to undo this change if you want to update the firmware, or if you want to install or uninstall other packages using the update method.
[edit] Debian in a chroot

After enabling ssh, you can put a debian-image-file to the vfat-partition (where the documents reside..), loop-mount and chroot to it. Here is a HowTo
[edit] Connecting to eduroam

When trying to connect to an eduroam network the Kindle NT 4 typically throws an error message saying it cannot connect to enterprise networks. Follow the steps listed below to make it work anyhow. The procedure is basically just a merger of prior work from David Antoš and Andrew Beresford.

1. Leave the proximity of any eduroam networks

2. On a router under your control with WPA-PSK authentication and internet access create a network named eduroam

3. Join the fake eduroam network with your kindle

4. Turn off the router with the fake eduroam network

5. Connect to your kindle via usbnetwork/ssh (see above for details)

$ ssh kindle
#################################################
#  N O T I C E  *  N O T I C E  *  N O T I C E  # 
#################################################
Rootfs is mounted read-only. Invoke mntroot rw to
switch back to a writable rootfs.
#################################################

6. Make the root partition writable and create a directory for storing the required files

# mntroot rw              
system: I mntroot:def:Making root filesystem writeable
/dev/mmcblk0p1 on / type ext3 (rw,noatime,nodiratime)
# mkdir -p /usr/local/wpa-enterprise-hack

7. Download the PEM certificate for your institution. You might get away by fetching AddTrust_External_Root.pem from a standard linux distribution. I actually had to download deutsche-telekom-ca-2.pem for accessing eduroam in Germany but that may differ depending on your institution. Rename the certificate to certificate.pem and copy it to the freshly created folder.

$ scp certificate.pem kindle:/usr/local/wpa-enterprise-hack

8. Create a script called wpa_config.sh in /usr/local/wpa-enterprise-hack with the following content replacing user@institution.org with the e-mail address at your university or research facility and 123456789 with your password. The script patches the eduroam settings upon each reboot (yes that may seem like a weird approach, but it's actually necessary)

#!/bin/sh

sleep 6

id="`wpa_cli list_networks | grep eduroam | cut -f1 | sed -n '1p'`"

exec="`wpa_cli << EOF
set_network $id ssid \"eduroam\"
set_network $id scan_ssid 1
set_network $id key_mgmt WPA-EAP
set_network $id pairwise TKIP
set_network $id group TKIP
set_network $id eap PEAP
set_network $id identity \"user@institution.org\"
set_network $id password \"123456789\"
set_network $id phase1 \"peaplabel=0\"
set_network $id phase2 \"auth=MSCHAPV2\"
set_network $id ca_cert \"/usr/local/wpa-enterprise-hack/certificate.pem\"
enable_network $id
quit
EOF
`"
echo $exec

9. Make wpa_config.sh executable

# chmod u+x /usr/local/wpa-enterprise-hack/wpa_config.sh

10. Make a backup of the wpa_supplicant

# cd /etc/init.d
# cp wpa_supplicant wpa_supplicant-orig

11. Extend the do_start() section of /etc/init.d/wpa_supplicant with a call to wpa_config.sh.

do_start () {
(…)
start-stop-daemon -o -S -b -x $WPA_SUP_BIN -- $WPA_SUP_CONF "$@"
msg "wpa supplicant started" I
/usr/local/wpa-enterprise-hack/wpa_config.sh &
(…)
}

12. Make the root filesystem read-only again and reboot your kindle.

# mntroot ro
system: I mntroot:def:Making root filesystem read-only
/dev/mmcblk0p1 on / type ext3 (ro,noatime,nodiratime)
# shutdown -r now
Broadcast message from root (pts/0) (Thu Jan  2 20:06:04 2014):
The system is going down for reboot NOW!

13. If you now enter the proximity of an eduroam network the kindle should automatically connect (may take 5-10 seconds).
[edit] Debricking / Un-demoing / Flashing firmware

The recommended tool for debricking and un-demoing Kindle 4 is a Linux LiveCD/LiveUSB called Kubrick.

For other possibilities and information take a look into the "Simple debricking..." thread. 




------------------------

https://www.sven.de/kindle/

fiona65b
