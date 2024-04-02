# Fix issues with Realtek NIC on pfSense 2.6.0 (Potentially)
Howdy!

Today I will share some of my experience updating to pfSense 2.6.0 and the aftermath regarding my Realtek NIC's. It is generally known that Realtek NIC's can cause (quite random) issues on pfSense. My experience with 2.6.0 was not different.

tldr; scroll down to see the steps I performed to resolve the problems.

I recently got myself a quite bargain Minisforum mini PC with 8 GB DDR4 and an Intel(R) Celeron(R) J4125 CPU @ 2.00GHz. Although cheap, the only downside I saw was the Realtek NIC's but took the gamble anyways. Installed pfSense 2.5.0 and all was well, no issues to be spotted. Noice!

A week+ or so later, pfSense 2.6.0 came out. I upgraded. Again all seemed to be fine for about a day or 5. Suddenly the WAN interface started flapping. It brought everything down with it (any network connectivity/web/ssh -service was no longer reachable). A hard reboot 'fixed' it sometimes for a day, sometimes an hour, sometimes 5 minutes. Very random. After doing some extensive googling I found that in pfSense 2.6.0 they switched to the updated FreeBSD re(4) driver. Aha! Seems like a possible culprit then.

Luckily there are official Realtek drivers, available as a kmod pkg. Time to give it a try!

To install the Realtek kmod pkg, open a shell (via ssh) and follow the below:

Step 1: Download and install the package:

fetch -v https://pkg.opnsense.org/FreeBSD:12:amd64/snapshots/latest/All/realtek-re-kmod-196.04.txz
pkg install -f -y realtek-re-kmod-196.04.txz

Step 2: Make sure the changes apply on boot:

Edit (create) the file: /boot/loader.conf.local and insert the following 2 lines:

if_re_load="YES"
if_re_name="/boot/modules/if_re.ko"

Save the file and reboot your device.

Note: After installation they suggest to add the lines to /boot/loader.conf. However, this file might get recompiled with certain changes in pfSense. Hence use the .local extension so this wont be an issue.

Step 3: Verify the drivers are loaded

Again, go into a shell and execute the command: kldstat
This should output a list, now containing an entry called if_re.ko

if this is not listed, then the driver has not been loaded for some reason. I am not covering troubleshooting of that here.

That's it! Now see if your problems are resolved.

For those that just updating the driver did not fix the issue (as my WAN would drop every 24 hours even after the driver update):

I first added the following lines to /boot/loader.conf

hw.re.msi_disable=1
hw.pci.enable_msix=0
hw.pci.enable_msi=0

This did not resolve the issue for me, so I found a post that suggested:

Go to System / Advanced / Networking and tic "Disable hardware checksum offload"

I have had ZERO issues since!
