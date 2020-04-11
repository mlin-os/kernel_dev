### Tasks
- [ ] Install Linux;
- [ ] Setup environment for kernel development;

#### How to learn Linux kernel development?
Here are some answers copied from Quora:

> Find a very simple component in the kernel and draw a diagram of what it is doing. Several diagrams, possibly. You'll want a flow chart but you'll also want to show how data changes.
> Don't chase down too many calls, you want to stay fairly local. Treat most calls as entry ways into black boxes. You know what goes in and something of what comes out.
> If you get stuck because you don't have information, then expand the relevant black box in another couple of diagrams, as far as needed.
> You now have the hang of things. Find something that actually interests, repeat until you think you understand it, then tweak and see what happens. Experiment.
> Once you're happy, copy the latest tree from git into your own github account, copy to your computer, make changes that appeal to you and send that back to your github account once you're sure you've checked for bugs and that the coding guidelines are followed. Once it's there, join the linux kernel developer’s mailing list and follow the guidelines carefully.

### Problems unsolved
1. Fix acpi error
2. 

### Create an enviroment for kernel development
#### Install Debian 10
1. Make a free partition for Debian[^1]

Use Windows+R hotkey to open Run window. Then type "Diskmgmt.msc" and click "OK" or hit "Enter" key.

2. Download Debian 10 from debian.org

Here I choose Debian testing distribution from [debian.org](https://cdimage.debian.org/cdimage/weekly-builds/amd64/iso-dvd/) by [Free Download managers](https://cdimage.debian.org/cdimage/weekly-builds/amd64/iso-dvd/). Debian testing version will have 5.4 linux kernel[^2].

3. Create a bootable USB[^3]

You need at lease 4GB for 32-bit or 8GB for 64-bit system on USB. Here I choose [rufus](https://rufus.ie) instead of [UNetbootin](https://unetbootin.github.io) to create a bootable use for its speed and easy of use. 

4. Switch to Boot from the USB stick[^4]

Restart your computer and press a special function key (usually F10, F12 or F9 etc, depending on the vendor specifications, Here mine is F10).

5. Install Debian

It's the simplest part.

6. Solve missing non-free fireeware problems

realtek-firmware: missing rtl8125a-3.fw, see [this](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=947356) for help.
```Bash
root# cd /lib/firmware/rtl_nic
root# wget https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/rtl_nic/rtl8125a-3.fw
```

7. Solve possible missing firmware for r8169

```Bash
sudo apt-get install firmware-realtek
```

8. Solve possible missing firmware for non-free module

```Bash
sudo apt-get install firmware-linux-nonfree
```

#### Install 

### FAQ
1. How to remove grub2 if remove dual system?

See [diskpart](https://askubuntu.com/questions/429610/uninstall-grub-and-use-windows-bootloader) for help. It's awesome.

2. How to install wifi adapter if wifi adapter is not found?

First, check what wifi adapter you use by `lspic` command[^5].
```Bash
lspci | grep -i wireless
```
Then, go to vendor's homepage and download firmware related to your device. My device is Intel® Wireless-AC 9560, the firmware of which can be found from [intel](https://www.intel.com/content/www/us/en/support/articles/000005511/network-and-i-o/wireless-networking.html).

Further, if want to check network related
```Bash
lspci | grep -i --color -E 'network|ethernet'
```
see [this](https://www.cyberciti.biz/faq/linux-list-network-cards-command/) for more help.

3. `sudo` problem

Add the user to the [sudo](https://askubuntu.com/questions/7477/how-can-i-add-a-new-user-as-sudoer-using-the-command-line) group.
```bash
/sbin/usermod -aG sudo mu
```

4. The shortcut of terminal is missing

Do as [this configure](https://unix.stackexchange.com/questions/41283/how-to-run-the-terminal-using-keyboard-shortcuts-in-gnome-2) does.

5. `sudo apt-get update` failed

It's caused by sources.list probelm. Revise your source as [this manual](http://forums.debian.net/viewtopic.php?f=5&t=143325) does. The nearest ftp server of Debian can be found [here](https://www.debian.org/mirror/list).
```bash
sudo vi /etc/apt/sources.list
deb http://ftp2.cn.debian.org/debian/ testing main contrib non-free
deb http://ftp2.cn.debian.org/debian/ testing-updates main contrib non-free
```

6. Fix "ACPI BIOS Error" - AE_ALREADY_EXISTS

Try updating [UEFI](https://support.hp.com/us-en/document/c00007682) as [this question](https://askubuntu.com/questions/1150029/random-system-hang-when-booting-ubuntu-19-04-with-kernel-5-0-0) suggests. But it FIALED!!!

7. How to access to command `sysctl`

Try Accessing the binary with the full path `/sbin/sysctl`. see [this blog](http://pkgs.loginroot.com/errors/notFound/sysctl) for details.

8. How to cut, copy and paste in the terminal[^6]?
To __cut__ Ctrl + Shift + X.
To __copy__ Ctrl + Shift + C.
To __Paste__ Ctrl + Shift + V.

9. Check your favarite alternative to vi
StackExchange gives [the answer](https://vi.stackexchange.com/questions/3580/how-do-i-tell-if-vi-or-vim-is-installed-on-my-linux-distribution):
```Bash
update-alternatives --list vi
```

10. Update nvidia display card driver
identify your [graphic card](https://wiki.debian.org/NvidiaGraphicsDrivers)
```Bash
lspci -nn | egrep -i "3d|display|vga"
```

display default display manager
```Bash
sudo cat /etc/X11/default-display-manager
/usr/sbin/gdm3
```

11. How to identify a device

see [this](https://wiki.debian.org/HowToIdentifyADevice) for help.

### History
2020/03/06 - Read HOWTO do Linux kernel development from kernel.org;
2020/03/08 - Download Debian 10 from debian.org and make bootable usb;
2020/03/09 - Solve the grub remenant problem after remove debian;
2020/03/10 - Solve the wifi adpater not found problem and `sudo apt-get update` failed problem;
2020/03/11 - Do a summary;
2020/03/12 - 1. Try fix the acpi bugs by updating UEFI from HP homepage. But it's too slow!!! 2. And do reading about memory management.
2020/03/21 - 1. Install Outline VPN;
2020/03/24 - continue to solve missing fireware problem;

### Reference
[^1]:https://www.diskpart.com/windows-10/windows-10-disk-management-0528.html
[^2]:https://en.wikipedia.org/wiki/Debian_version_history
[^3]:https://www.maketecheasier.com/use-rufus-create-bootable-flash-drive/
[^4]:https://askubuntu.com/questions/1031993/how-to-install-ubuntu-18-04-alongside-windows-10
[^5]:https://www.cyberciti.biz/faq/linux-find-wireless-driver-chipset/
[^6]:https://www.fosslinux.com/14088/how-to-copy-and-paste-commands-in-linux-terminal.htm