### Preamble

#### How to learn Linux kernel development?
Here are answers copied from Quora:

> Find a very simple component in the kernel and draw a diagram of what it is doing. Several diagrams, possibly. You'll want a flow chart but you'll also want to show how data changes.
> Don't chase down too many calls, you want to stay fairly local. Treat most calls as entry ways into black boxes. You know what goes in and something of what comes out.
> If you get stuck because you don't have information, then expand the relevant black box in another couple of diagrams, as far as needed.
> You now have the hang of things. Find something that actually interests, repeat until you think you understand it, then tweak and see what happens. Experiment.
> Once you're happy, copy the latest tree from git into your own github account, copy to your computer, make changes that appeal to you and send that back to your github account once you're sure you've checked for bugs and that the coding guidelines are followed. Once it's there, join the linux kernel developer’s mailing list and follow the guidelines carefully.

#### Good tutorials
1. [FirstKernelPatch](https://kernelnewbies.org/FirstKernelPatch)
2. [get stable kernel source code](https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/)

#### Reading List

1. [Understanding Linux Kernel Stack](https://wenboshen.org/posts/2015-12-18-kernel-stack.html)

### Tasks
- [x] Install Linux;
- [x] Setup environment for kernel development;
- [x] Compiler kernel;
- [x] Replace current kernel;
- [ ] Write a module;
- [ ] Debug the kernel;
- [ ] Trace the mail list;
- [ ] Find an interesting and soluable question;

### Problems unsolved

2. Download the latest kernel directly from [github](https://github.com/torvalds/linux.git) failed

3. More efficiently use of vim

4. Should I maintain my own PGP?

see [kernel doc](https://www.kernel.org/doc/html/v4.16/process/maintainer-pgp-guide.html) for details. 


### Install Debian 10

Before you start to install debian, I suggest you to choose a relatively well-supported computer by the debian community or see other release such as Ubuntu. It will absolutely get you rid of many uneccessary faults and errors.

Based on my personal experience, I recommend Dell XPS 15. Here are some infos you can read on this [topic](https://wiki.debian.org/InstallingDebianOn).

1. Make a free partition for Debian[^1]

Use Windows+R hotkey to open Run window. Then type "Diskmgmt.msc" and click "OK" or hit "Enter" key. Based on my personal experience, I recommend to assign 150G free space at least, maybe 200G is better (I assign 250G instead). Because the kernel project is big, you may have multiple images in your system. I used to have memory shortage warning when I assign 100G space for debian.

2. Download Debian 10 from debian.org

Here I choose Debian testing distribution from [debian.org](https://cdimage.debian.org/cdimage/weekly-builds/amd64/iso-dvd/) by [Free Download managers](https://cdimage.debian.org/cdimage/weekly-builds/amd64/iso-dvd/). Debian testing version will have 5.4 linux kernel[^2].

3. Create a bootable USB[^3]

You need at lease 4GB for 32-bit or 8GB for 64-bit system on USB. Here I choose [rufus](https://rufus.ie) instead of [UNetbootin](https://unetbootin.github.io) to create a bootable use for its speed and easy of use. 

4. [Do important settings in BIOS](https://wiki.archlinux.org/index.php/Dell_XPS_15_7590)

First, restart your computer and press a special function key (usually F10, F12 or F9 etc, depending on the vendor specifications, Here mine is F2 on XPS 15) to enter in BIOS.

Second, under 'System Configuration', change the SATA Mode from the default "RAID" to "AHCI". This will allow Linux to detect the NVME SSD.

Third, under 'POST Behaviour', change "Fastboot" to "Thorough". This prevents intermittent boot failures.

5. Install Debian

It's the simplest part.

6. Fix windows problem

After finishing the above operations, I have installed Debian successfully. However, when I switch to windows, I get below issue:

> inaccessible boot device (Windows blue screen), followed by BitLocker recovery key prompt that leads to system repair that fails.

Follow this [guide](https://triplescomputers.com/blog/uncategorized/solution-switch-windows-10-from-raidide-to-ahci-operation/) to close your safeboot.

7. Setup the update repo of apt

It's caused by sources.list problem. Revise your source as [this manual](http://forums.debian.net/viewtopic.php?f=5&t=143325) does. The nearest ftp server of Debian can be found [here](https://www.debian.org/mirror/list).
```Bash
sudo vi /etc/apt/sources.list
deb http://ftp2.cn.debian.org/debian/ testing main contrib non-free
deb http://ftp2.cn.debian.org/debian/ testing-updates main contrib non-free
```

8. Solve possible missing firmwares non-free module

```Bash
sudo apt-get install firmware-linux-nonfree
```

### Install other useful tools

1. Useful tool list

    * git            - necessary
    * ctags          - create tags file
    * cscope         - similar to ctags but more powerful
    * graphviz       - plot tool
    * outline        - vpn
    * sougoupinyin   - failed
    * exiftool       - tool to manage meta info
    * zotero         - A tool to manage article reference

2. Install and Config git

git-scm gives a [basic setup](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup)
```Bash
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
$ git config --global core.editor vim
```

Instead of use ssh which must manager your key, here I choose store the key with [credentials](https://stackoverflow.com/questions/35942754/how-to-save-username-and-password-in-git-gitextension)
```Bash
# save the key in cache for 3600s
git config --global credential.helper 'cache --timeout=3600'
```
You can also save it permenantly
```Bash
git config --global credential.helper store
```

3. Install and Config vim

[vimawesome](https://vimawesome.com/)  is a vim plugin web where you can search and install vim plugin. you can find the most popular and powerful ones recommended by other users.

Here are some useful plugins that I have tried:
| Name              | Description                | Source                                                 |
|-------------------|----------------------------|--------------------------------------------------------|
| vim-plug          | one of vim plugin managers | https://github.com/junegunn/vim-plug.git               |
| molokai           | color scheme               | https://github.com/tomasr/molokai.git                  |
| nerdtree          | directory explorer tool    | https://github.com/junegunn/vim-plug.git               |
| tarbar            | search and jump tag tool   | https://github.com/majutsushi/tagbar.git               |
| vim-markdown      | vim config for markdown    | https://github.com/plasticboy/vim-markdown.git         |
| vim-indent-guides | show vim indent level      | https://github.com/nathanaelkane/vim-indent-guides.git |
| vim-table-mode    | fast way to create table   | https://github.com/dhruvasagar/vim-table-mode.git      |

4. Install typora

Copy below [info](https://typora.io/#linux) from typora homepage.
```Bash
wget -qO - https://typora.io/linux/public-key.asc | sudo apt-key add -

# add Typora's repository
sudo add-apt-repository 'deb https://typora.io/linux ./'
sudo apt-get update

# install typora
sudo apt-get install typora
```

5. Install exiftool

As this [post](https://www.poftut.com/how-to-install-and-use-exiftool-in-linux-windows-kali-ubuntu-mint-with-examples/) says
```Bash
sudo apt-get install libimage-exiftool-perl
```

use exiftool to [remove xmp metadata](https://exiftool.org/forum/index.php?topic=6498.0) from pdf
```Bash
exiftool -all:all= XXX.pdf
```

6. Install libpinyin

Here I choose English as the system default language, so I need to install chinese input method. Because sougoupinyin need qt4 which does not exist in the latest debian system, I choose libpinyin instead as [debian wiki](https://wiki.debian.org/gnome-chinese-input) suggested.

First, add chinese locales support:
```Bash
sudo dpkg-reconfigure locales
```
Add zh_CN.UTF-8 by hitting space stroke.

Second, install fcitx and libpinyin
```Bash
sudo apt-get install fcitx fcitx-libpinyin
```

Third, config fcitx, choose fcitx method in
```Bash
im-config
```

Fourth, add libpinyin in [input method](https://silentming.net/blog/2015/11/16/add-chinese-in-english-debian/)
```
fcitx-configuretool
```

By the way, you can use `fcitx-diagnose` tool to check what's missing.

### Build Linux Kernel
#### Display current kernel version
```Bash
# First way
mu@ustc:~$ uname -r
5.6.0-1-amd64

# Second way
mu@ustc:~$ cat /proc/version
Linux version 5.6.0-1-amd64 (debian-kernel@lists.debian.org) (gcc version 9.3.0 (Debian 9.3.0-11)) #1 SMP Debian 5.6.7-1 (2020-04-29)

# Third way
mu@ustc:~$ hostnamectl | grep Kernel
            Kernel: Linux 5.6.0-1-amd64

# Fourth way
mu@ustc:~$ sudo dmesg |grep 'Linux'
[sudo] password for mu:
[    0.000000] Linux version 5.6.0-1-amd64 (debian-kernel@lists.debian.org) (gcc version 9.3.0 (Debian 9.3.0-11)) #1 SMP Debian 5.6.7-1 (2020-04-29)
```

#### How to upgrade Linux kernel

Debian homepage provide this [method](https://wiki.debian.org/HowToUpgradeKernel).

First, detect alternative kernel images by
```
apt-cache search linux-image
```

Then, install using
```Bash
$ sudo apt install linux-image-<flavour>
```

#### Build Debian latest kernel from source code

Here is a detailed [process](https://www.dwarmstrong.org/kernel/) from a kernel developer. Or do as the [debian homepage](https://kernel-team.pages.debian.net/kernel-handbook/ch-common-tasks.html#s-common-official) demenstrates

1. Get the debian latest kernel source code

```Bash
# apt-get install linux-source-5.6
$ tar xaf /usr/src/linux-source-5.6.tar.xz
```

2. How to detect the version of linux kernel source tree?
```Bash
mu@ustc:~/linux-source-5.6$ make kernelversion
5.6.7
```

2. Get the config file of your current running kernel

As this [question](https://superuser.com/questions/287371/obtain-kernel-config-from-currently-running-linux-system) answers:
```Bash
cp /boot/config-$(uname -r) your_kernel_source_top_dir/.config
```

3. Moidify the copied configuration and set

```Bash
CONFIG_SYSTEM_TRUSTED_KEYS = ""
```

4. Build the kernel

```Bash
make clean
make deb-pkg LOCALVERSION=-custom
```

5. New packages will be stored at

```Bash
mu@ustc:~/linux-source-5.6$ ls ../*deb
../linux-headers-5.6.7-custom_5.6.7-custom-1_amd64.deb
../linux-image-5.6.7-custom_5.6.7-custom-1_amd64.deb
../linux-image-5.6.7-custom-dbg_5.6.7-custom-1_amd64.deb
../linux-libc-dev_5.6.7-custom-1_amd64.deb
```

6. Install and reboot

```Bash
sudo dpkg -i ../linux-image-5.6.7-custom-dbg_5.6.7-custom-1_amd64.deb
sudo shutdown -r now
```

#### Build kernel from kernel source distributed from www.kernel.org

1. Get the latest stable kernel source code

```Bash
wget https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.6.14.tar.xz 
wget https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.6.14.tar.sign 
```

2. [Use the web key directory](https://www.kernel.org/signature.html)

```Bash
gpg2 --locate-keys torvalds@kernel.org gregkh@kernel.org
```

3. Verify the signature

```Bash
mu@ustc:~/linux$ unxz -c linux-5.6.14.tar.xz |gpg --verify linux-5.6.14.tar.sign -
gpg: Signature made Wed 20 May 2020 02:23:42 PM CST
gpg:                using RSA key 647F28654894E3BD457199BE38DBBDC86092693E
gpg: Good signature from "Greg Kroah-Hartman <gregkh@kernel.org>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 647F 2865 4894 E3BD 4571  99BE 38DB BDC8 6092 693E
```

4. Unpack the source code

```Bash
tar xaf linux-5.6.14.tar.xz
```

5. Build and replace your kernel

The remain tasks are the same as building debian kernel.

#### The other way to build kernel

see [this guide](https://kernelnewbies.org/OutreachyfirstpatchSetup?action=show&redirect=OPWfirstpatchSetup) from kernelnewbies.org.

#### Enable KDB tool

1. the relationship of KGDB and KDB

> They are two different debugger front ends.
> Kdb is simplistic shell-style interface which you can use on a system console with a ekyboard or serial console. Kdb is not a source lvel debugger.
> Kgdb is intended to be used as a source level debugger for the Linux kernel.
> It's possible to use either of them and dynamically transit between them if you configure the kernel properly at compile and runtime.

### FAQ
1. How to remove grub2 if remove dual system?

See [diskpart](https://askubuntu.com/questions/429610/uninstall-grub-and-use-windows-bootloader) for help. It's awesome.
> 1. Run a `cmd.exe` process with administrator privileges
> 2. Run `diskpart`
> 3. Type: `list disk` then `sel disk X` where X is the drive your boot files reside on
> 4. Type `list vol` to see all partitions (volumes) on the disk (the EFI volume will be formatted in FAT, others will be NTFS)
> 5. Select the EFI volume by typing: `sel vol Y` where Y is the `SYSTEM` volume (this is almost always the EFI partition)
> 6. For convenience, assign a drive letter by typing: `assign letter=Z:` where Z is a free (unused) drive letter
> 7. Type `exit` to leave disk part
> 8. While still in the cmd prompt, type: `Z:` and hit enter, where Z was the drive letter you just created.
> 9. Type `dir` to list directories on this mounted EFI partition
> 10. If you are in the right place, you should see a directory called `EFI`
> 11. Type `cd EFI` and then `dir` to list the child directories inside `EFI`
> 12. Type `rmdir /S ubuntu` to delete the ubuntu boot directory

2. How to install wifi adapter if wifi adapter is not found?

First, check what wifi adapter you use by `lspic` command[^5].
```Bash
mu@ustc:~$ lspci |grep -i -E 'wi-?fi'
3a:00.0 Network controller: Intel Corporation Wi-Fi 6 AX200 (rev 1a)
```

Then, go to vendor's homepage and download firmware related to your device. My device is Intel® Wi-Fi 6 AX200 160MHz, the firmware of which can be found from [intel](https://www.intel.com/content/www/us/en/support/articles/000005511/network-and-i-o/wireless-networking.html).

Further, if want to check network related
```Bash
lspci | grep -i --color -E 'network|ethernet'
```
see [this](https://www.cyberciti.biz/faq/linux-list-network-cards-command/) for more help.

3. `sudo` problem

Add the user to the [sudo](https://askubuntu.com/questions/7477/how-can-i-add-a-new-user-as-sudoer-using-the-command-line) group.
```Bash
/sbin/usermod -aG sudo mu
```

4. The shortcut of terminal is missing

Do as [this configure](https://unix.stackexchange.com/questions/41283/how-to-run-the-terminal-using-keyboard-shortcuts-in-gnome-2) does.

5. Fix "ACPI BIOS Error" - AE_ALREADY_EXISTS

Try updating [UEFI](https://support.hp.com/us-en/document/c00007682) as [this question](https://askubuntu.com/questions/1150029/random-system-hang-when-booting-ubuntu-19-04-with-kernel-5-0-0) suggests. But it FIALED!!!

6. How to access to command `sysctl`

Try Accessing the binary with the full path `/sbin/sysctl`. see [this blog](http://pkgs.loginroot.com/errors/notFound/sysctl) for details.

7. How to cut, copy and paste in the terminal[^6]?

To __cut__ Ctrl + Shift + X.

To __copy__ Ctrl + Shift + C.

To __Paste__ Ctrl + Shift + V.

8. Check your favarite alternative to vi

StackExchange gives [the answer](https://vi.stackexchange.com/questions/3580/how-do-i-tell-if-vi-or-vim-is-installed-on-my-linux-distribution):
```Bash
update-alternatives --list vi
```

9. Update nvidia display card driver

identify your [graphic card](https://wiki.debian.org/NvidiaGraphicsDrivers)
```Bash
lspci -nn | egrep -i "3d|display|vga"
```

display default display manager
```Bash
sudo cat /etc/X11/default-display-manager
/usr/sbin/gdm3
```

10. How to identify devices that are connected to your computer

see [this](https://wiki.debian.org/HowToIdentifyADevice) for help.

11. Get [maximize/minimize/close button](https://askubuntu.com/questions/651347/how-to-bring-back-minimize-and-maximize-buttons-in-gnome-3) back

```Bash
gsettings set org.gnome.desktop.wm.preferences button-layout "close,minimize,maximize:"
```

12. How to list installed packages

This [blog](https://linuxize.com/post/how-to-list-installed-packages-on-debian/) gives below command
```Bash
sudo apt list --installed
```

13. How to undo integration of outline with linux system?

Here is a method copied from [jigsaw](https://github.com/Jigsaw-Code/outline-client/issues/648):
```Bash
rm -rf ~/.config/Outline/
rm -f ~/.config/autostart/Outline-Client.AppImage.desktop
find ~/.local/share/icons -name "appimagekit-outline-client.png" -delete
rm -f /path/to/Outline-Client.AppImage
```

### History
| Date       | Info                                                                                                                        |
|------------|-----------------------------------------------------------------------------------------------------------------------------|
| 2020/03/06 | Read HOWTO do Linux kernel development from kernel.org;                                                                     |
| 2020/03/08 | Download Debian 10 from debian.org and make bootable usb;                                                                   |
| 2020/03/09 | Solve the grub remenant problem after remove debian;                                                                        |
| 2020/03/10 | Solve the wifi adpater not found problem and `sudo apt get update` failed problem;                                          |
| 2020/03/11 | Do a summary;                                                                                                               |
| 2020/03/12 | 1. Try fix the acpi bugs by updating UEFI from HP homepage. But it's too slow!!! 2. And do reading about memory management. |
| 2020/03/21 | 1. Install Outline VPN;                                                                                                     |
| 2020/03/24 | Continue to solve missing fireware problem;                                                                                 |
| 2020/04/10 | Install sougoupinyin failed, use libpinyin instead, because testing doesn't have qt4 which existing in jessie;              |
| 2020/04/11 | Install typora;                                                                                                             |
| 2020/04/29 | Give a representation of slub allocator;                                                                                    |
| 2020/05/01 | Do a summary;                                                                                                               |
| 2020/05/05 | Compile kernel as debian homepage suggested;                                                                                |
| 2020/05/06 | Get config file of your current running kernel;                                                                             |
| 2020/05/12 | Update vim plugins recommendation list;                                                                                     |
| 2020/05/21 | Replace current kernel using source code distributed from www.kernel.org                                                    |
| 2020/08/31 | Switch to Dell XPS 15 and redo above works                                                                                  |

### Reference
[^1]:https://www.diskpart.com/windows-10/windows-10-disk-management-0528.html
[^2]:https://en.wikipedia.org/wiki/Debian_version_history
[^3]:https://www.maketecheasier.com/use-rufus-create-bootable-flash-drive/
[^4]:https://askubuntu.com/questions/1031993/how-to-install-ubuntu-18-04-alongside-windows-10
[^5]:https://www.cyberciti.biz/faq/linux-find-wireless-driver-chipset/
[^6]:https://www.fosslinux.com/14088/how-to-copy-and-paste-commands-in-linux-terminal.htm
[^7]:https://kernel-team.pages.debian.net/kernel-handbook/ch-common-tasks.html#s-common-official
