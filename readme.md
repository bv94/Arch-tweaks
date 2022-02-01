# Arch Linux Tweaks

### Table of Contents

[Reflector Mirrors](#reflector-mirrors) <br>
[Conifgure Pacman](#configure-pacman) <br>
[Makepkg Tweaks](#makepkg) <br>
[Installing Yay](#yay) <br>
[Installing Pamac](#pamac) <br>
[Performance Tweaks](#performance) <br>
[Installing Sound Drivers and Improving Sound Quality](#sound) <br>
[Installing Video/Image Codecs](#video-image) <br>
[Adding Filesystem Support](#filesystem) <br>
[Installing Fonts](#fonts) <br>
[Setting I/O Schedulers](#io) <br>
[Network Tweaks](#network) <br>
[Initramfs Tweaks](#initramfs) <br>
[Laptop battery Improvement](#laptop) <br>
[Fusuma Touchpad Gestures](#touchpad) <br>
[Donations](#donations) <br>

Welcome to my Arch Linux Tweaks Guide, this is a personal collection of tweaks and settings that I do to a fresh Install on my system to get it running the way I like it to. I won’t be covering things such as microcodes in this guide as they are covered in my Arch Linux install guide already and this guide will be focusing on the next steps directly from there.


Alot of people may prefer to create a bunch of dotfiles in there home directory to allow different users to have different settings for things like pacman, audio, makepkg settings etc. I see no need for this the users are on the same system as you so why not set them as system wide settings? To each there own I have listed both methods where it applies if you choose to do it one way over the other.

## Reflector Mirrors <a name="reflector-mirrors"></a>

Lets start with getting yours mirror configured this will make sure you are downloading packages from locations closest to you so you achieve the best download speed. In my Arch Install Guide we have already installed the reflector package we need and just need to set up reflector and enable the systemd service. First run this command

`sudo reflector -c CountryName --sort rate --save /etc/pacman.d/mirrorlist`

Change the CountryName to your Country Name inside quotes '' and it should give you just mirrors for your country. Next we will enable the systemd service by running

`sudo systemctl enable reflector.timer`

and that’s it now it will reresh your mirrors every bootup. If you are really tight on resources and don’t travel much or have a desktop you can skip running the service as the mirrors aren’t going to change much.

## Configure pacman <a name="configure-pacman"></a>

Now we can configue the pacman configuration file to enable parallel downloads by running

`sudo nano /etc/pacman.conf`

and scroll down to to Misc options and uncomment (remove the #) on ParallelDownloads =5 I also change this number to 10 as I have a pretty solid connection and can handle downloading more things at a time. After that scroll all the way down to the bottom and below where the commented Server is we can start adding extra repos to download pre compiled packages to save time instead of compiling them ourselves. I like to add the following repo as it offers pre compiled AUR packages

```
[chaotic-aur]
Include = /etc/pacman.d/chaotic-mirrorlist
```

Then we can do Control + X then press y to save and enter after that run

`sudo pacman -Syyu`

to update the repos and check for updates. Due note that some of these repos are from unsigned parties so add them at your own safety risk. I switch a lot of my drivers over to git versions however won’t be covering that in this guide as I don’t recommend others doing to so as some updates can cause issues that you will need to fix.

## Makepkg Tweaks <a name="makepkg"></a>

Although we haven’t really have to build anything on our own system yet in this guide we will need to and it would be nice if we could use the full proccessing power of our system for those tasks as they can take awhile especially the kernel. First we need to know how many cpu threads you have on your system we can check this by running

`nproc`

keep the number it tells you in mind we will need that for later now we can edit the makepkg file by running

`sudo nano /etc/makepkg.conf`

or make it a user only setting by running

```
sudo cp /etc/makepkg.conf ~/.makepkg.conf
sudo nano ~/.makepkg.conf
```

Once inside the makepkg.conf file scroll down till you come across “Compiler and Linker Flags” in this section we are looking for the CF Flags option make sure its uncommented (remove # infront if there is one) and and change -march and -mtune to native so it looks like this

`-march=native -mtune=native`

Adding this will just add architecture-specific optimizations to compiling.Now we can go down a bit more to RUSTFLAGS and after opt-level=2 add “-C target-cpu=native” after it within the same quotations and uncomment the line (remove the # in front of RUSTFLAGS) so it looks like this

`RUSTFLAGS="-C opt-level=2 -C target-cpu=native"`

Again same thing as the last line for RUST compiles. Now lets go down a bit more and until you will see MAKEFLAGS uncomment this line and change it to -j and then the number of threads on you have on your system plus 1 so on my 5950X it looks like this

`MAKEFLAGS="-j33"`

This will allow your system to use all the threads on the CPU to build packages. Next scroll all the way down to the next section “BUILD ENVIRONMENT” and and the last line before the next section uncomment BUILDDIR and make sure it goes to /tmp/makepkg like so

`BUILDDIR=/tmp/makepkg`

now scroll all the way down to “COMPRESSION DEFAULTS” and we need to change a few things starting with the first line COMPRESSGZ replace gzip with the word pigz like this

`COMPRESSGZ=(pigz -c -f -n)`

pigz works the same as gzip but sohuld use all the cores on your system when compressing to .gz

Now lets move down to the next line COMPRESSBZ2 this is another simple change all we need to do is add a p in front of bzip2 so it reads pbzip2 like this

`COMPRESSBZ2=(pbzip2 -c -f)`

just like the last change for .gz this will also enable all cores for .bz2 Now we can go the next line
COMPRESSXZ and after -z but before the final – you want to add --threads=0 like so

`COMPRESSXZ=(xz -c -z --threads=0 -)`

Threads being set to 0 will use all threads on the system for .xz or you can manually input the number of threads you got earlier from nproc.

For the last Compression method COMPRESSZST we are going to also add --threads=0 after -q and before the final – like so

`COMPRESSZST=(zstd -c -z -q --threads=0 -)`

Now this next option is option but we can scroll down to the next section called EXTENSION DEFAULTS and remove .gz from the end of PKGEXT this will speed up install and packaging your items but will cause you to have slightly larger file sizes. It should look like this

`PKGEXT='.pkg.tar'`

## Installing Yay <a name="yay"></a>

I use yay over the other options such as yaourt and paru. Yay is just easier for me to use I find and doesn’t ask me to review PKGBUILD Files like paru which just gets annoying in my opinion.
For those that don’t know what yay, yaourt or paru are these are called AUR Helpers they are similar to pacman and help you connect directly to the AUR. First we are going to use git which you should have already from my install guide if not just run

`sudo pacman -S git`

and then we can run

`git clone https://aur.archlinux.org/yay.git`

to clone the repo to our home folder, we can delete it again when we are finished with it we won’t need a special folder for it as yay can upgrade through the AUR once we have access to it. Now we can move into the folder and make the package by running

```
cd yay
makepkg -si
```

This will install yay for us and may install go along the way thats about it now we can go back to our home fodler by running

`cd ../`

and then we can delete the folder by running

`sudo rm -R yay`

## Installing pamac <a name="pamac"></a>

This might be a little bit different than how some other people in the Arch Community do it however I see no need for Flatpak or Snap package support so I install a version that doesn’t have that support built in. I will cover both versions but I recommend the first one.

To install Pamac without Flatpak or Snap Support run

`yay -S pamac-aur-git`

or install Pamac with Flatpak and Snap support run

`yay -S pamac-all-git`

-git usually means the latest/beta version but sometimes repos get out of date so you will have to check that on your own on a package by package basis. That’s pretty much it now in your Desktop if you go to your Applications and then under the Settings subdirectory you should see a new app called Add/Remove Software this is pamac. You going to want to open it up and go to prefrences under the sandwhich looking icon beside the minimize window button and under the tab AUR or Third Party make sure Enable AUR Support is Enabled. Under the General tab you can also adjust the parallel downloads and udner Advanced tab you can enable downgrade in case packlages break your system in the future and you need to roll back. If you downloaded the Flatpak and Snap Supported version make sure you go to those extra tabs in the settings and enable them also.

## Performance Tweaks <a name="performance"></a>

These are overall system tweaks some of which are used in the Garuda Linux Distro to increase gaming performance the first thing we are gonna do is download all the needed packages with

`yay -S ananicy-cpp irqbalance memavaild nohang preload prelockd uresourced`

Now explaining what each of these do is just going to make this list of tweaks insanely so instead what I'm going to do is provide the link below for each  
<b>ananicy-cpp</b> - https://gitlab.com/ananicy-cpp/ananicy-cpp<br>
<b>irqbalance</b> - https://github.com/Irqbalance/irqbalance<br>
<b>memavaild</b> - https://github.com/hakavlad/memavaild<br>
<b>nohang</b> - https://github.com/hakavlad/nohang<br>
<b>preload</b> - https://wiki.archlinux.org/title/Preload<br>
<b>prelockd</b> - https://github.com/hakavlad/prelockd<br>
<b>uresourced</b> - https://gitlab.freedesktop.org/benzea/uresourced<br>

Next we will enable these processes to startup at runtime using systemd and we can do that by running these commands

```
sudo systemctl disable systemd-oomd
sudo systemctl enable ananicy-cpp
sudo systemctl enable irqbalance
sudo systemctl enable memavaild
sudo systemctl enable nohang
sudo systemctl enable preload
sudo systemctl enable prelockd
sudo systemctl enable uresourced
```

I have also added the command to disable systemd-oomd because I don't use a swap partition as mentioned previously I would rather just buy more ram. However this is really disabled because nohang basically replaces this modules use and will prevent OOM (Out of Memory) errors.

Next we can run this

`sed -i 's|zram_checking_enabled = False|zram_checking_enabled = True|g' /etc/nohang/nohang.conf`

for those that don't understand this command this basically finds zram checking enabled and that changes that line to true.

Now we are going to make/edit a bunch of config files for performance even on my Framework Laptop these don't kill battery life to much but it is a difference so if that matters to you maybe skip all of these. We are starting with the cpu governor

`sudo nano /usr/lib/tmpfiles.d/cpu-governor.conf`

and paste

`w /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor - - - - performance`

Then like everything else in this guide with nano use Control + X to save and exit. Now we can set energy preferences to performance by opening

`sudo nano /usr/lib/tmpfiles.d/energy_performance_preference.conf`

and paste

`w /sys/devices/system/cpu/cpufreq/policy*/energy_performance_preference - - - - performance`

next pcie devices such as gpus

`sudo nano /usr/lib/tmpfiles.d/pcie_aspm_performance.conf`

and paste

`w /sys/module/pcie_aspm/parameters/policy - - - - performance`

next is another device power management setting

`sudo nano /usr/lib/tmpfiles.d/power_dpm_state.conf`

and paste

`w /sys/class/drm/card0/device/power_dpm_state - - - - performance`

If you have a newer AMD GPU then you can also open this file

`sudo nano /usr/lib/udev/rules.d/30-amdgpu-pm.rules`

and paste

`KERNEL=="card0", SUBSYSTEM=="drm", DRIVERS=="amdgpu", ATTR{device/power_dpm_state}="performance"`

If your running an older AMD GPU such as 300 series or older thne you may be running on the radeon driver and you can open this instead

`sudo nano /usr/lib/udev/rules.d/30-radeon-pm.rules`

and paste

`KERNEL=="card0", SUBSYSTEM=="drm", DRIVERS=="radeon", ATTR{device/power_dpm_state}="performance"`

Next we are going to set the SATA Drives to performance mode, If you use all NVMe (note that not all M.2 Drives are NVMe) drives like I do you don't need to set this I still do incase I need to connect to a SATA Drive though. We need to first run

`sudo nano /usr/lib/udev/rules.d/50-sata.rules`

and paste

```
# SATA Active Link Power Management
ACTION=="add", SUBSYSTEM=="scsi_host", KERNEL=="host*", ATTR{link_power_management_policy}="max_performance"
```

Lastly we are just going to set the hdparam rules. This wil cause your Internal and External HDDs to never turn off so if you use an external drive or have a HDD in your system thats a secondary drive you may want to not set this or unplug your external drive unless needed as it will probably cause the drive to die sooner just from never being able to sleep. If you use SSDs since they are using a flash NAND this is less of an issue unless your drives are overheating or something first we can open the file with

`sudo nano /usr/lib/udev/rules.d/69-hdparm.rules`

and paste

`ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="1", RUN+="/usr/bin/hdparm -B 254 -S 0 /dev/%k"`

## Installing sound drivers and Improving Sound Quality <a name="sound"></a>

In my install guide all I did was show you a basic Arch Install with a desktop but didn’t provide any sound to your system so we can add that now here so those with sound already you can skip this. We can install the packages for sound by running this command

```
yay -S pipewire-pulse pipewire-alsa pipewire-jack lib32-pipewire-jack alsa-plugins alsa-firmware sof-firmware alsa-card-profiles flac lib32-flac wavpack libdca libmad libvorbis lib32-libvorbis faad2 lib32-faad2 faac lib32-faac gst-plugins-good jack2 jack2-dbus
```

This will install every audio codec you should need unless you have a really specific use case and should give you the audio drivers you need for your system. If you just listen to a music streaming service such as Spotify you can remove the faac faad2 wavpack and flac packages as long as they have no other dependencies. Once you have these we can start to edit the config files to improve the sound quality. Changing these audio settings will increase CPU Usage on lower end cpus just keep that in mind and do your own settings before keeping these settings, however on my 3990X, 5950X and 1165G7 I don’t even notice a 1% hit on cpu usage so I assume it doesn’t really affect modern cpus unless your running a dual core maybe.
First we need to find out what byte order your cpu has and we can check that by running

`lscpu | grep 'Byte Order'`

Remember if it is Little Endian or Big Endian for later on. I like to edit the global system config by running

`sudo nano /etc/pulse/daemon.conf`

however you can also make a user specific config by running

```
cp /etc/pulse/daemon.conf ~/.config/pulse/daemon.conf
nano ~/.config/pulse/daemon.conf
```

This next step is mostly just uncommenting lines in this config a comment is made with ; instead of #. So we just need to delete the ; in front of the word for that setting to be applied. Im going to list in order the specific lines the need a comment removed and also the changes you need to make to some of the values and I will explain the changes after.

```
daemonize = no
cpu-limit = no
high-priority = yes
nice-level = -11
realtime-scheduling = yes
realtime-priority = 5
resample-method = soxr-vhq
avoid-resampling = false
enable-remixing = no
rlimit-rtprio = 9
default-sample-format = float32le
default-sample-rate = 96000
alternate-sample-rate = 48000
default-sample-channels = 2
default-channel-map = front-left,front-right
default-fragments = 2
default-fragment-size-msec = 125
```

If your system was Little Endian you can keep everything like this for now, however if you are running a Big Endian system you need to change float32le to flat32be. I have used a sample rate of 96KHz if you hear popping upon a reboot or no sound at all feel free to change this back to something like 48KHz you can also raise this to something like 192KHz if your audio setup supports it, below I can linked to more Sample rates if you have a higher end audio recording setup. If your having high latency play around with the default fragment and size msec settings.

Audio Sampling Rates - https://en.wikipedia.org/wiki/Sampling_(signal_processing)#Audio_sampling

Moving onto the ALSA Config now we can again edit the Global system config by running

`sudo nano /etc/asound.conf`

or make a user specific config by running

```
cp /etc/asound.conf ~/.asoundrc
nano ~/.asoundrc
```

If it errors copying the asound file don’t worry about it just means it doesn’t exist yet. If your file is empty you can just paste the following code otherwise if your file has text find “Use PulseAudio by default” and remove everything down to the final closing }. And replace it with the following

```
# Use PulseAudio plugin hw
pcm.!default {
   type plug
   slave.pcm hw
}
```

If your into Audio production also check out the app cadence which can provide some system checks for you - https://kx.studio/Applications:Cadence

Also don’t forget to switch to a realtime kernel for low latency. If your new to Audio Production on Linux and don’t know exactly what to use check out this page - https://jackaudio.org/applications/

Read more about upmixing/downmixing here - https://wiki.archlinux.org/title/Advanced_Linux_Sound_Architecture#High_quality_resampling

## Adding Video/Image Codecs <a name="video-image"></a>

These are just good to have never know when you will need them. Pretty simple just run the command to install and that’s it.

```
yay -S openjpeg libwebp libavif libheif dav1d rav1e daala-git libdv libmpeg2 libtheora libvpx x264 x265 xvidcore gstreamer gst-plugins-base gst-libav gstreamer-vaapi ffmpeg
```

## Adding Filesystem Support <a name="filesystem"></a>

Just like with Video/Image Codecs all we need to do is just run this command to add support

```
yay -S btrfs-progs dosfstools exfatprogs f2fs-tools e2fsprogs hfsprogs jfsutils nilfs-utils ntfs-3g xfsprogs apfsprogs-git bcachefs-tools zfs-utils
```

You may not need all of these like BTRFS, JFS or ZFS maybe even the Apple HFS and APFS options but I like to keep them installed so that Im ready to work with any drive I happen to need to interact with. Feel free to adjust this to your liking.

## Installing Fonts <a name="fonts"></a>

These are just extra fonts for some apps that manually set there own and also includes apple-fonts which gives you the SF Pro Fonts which I have set as my default font in Firefox and My Desktop.

`yay -S noto-fonts noto-fonts-cjk ttf-dejavu ttf-liberation ttf-opensans apple-fonts`

## Setting I/O Schedulers <a name="io"></a>

Linux can get very slow during I/O intense applications however this can be improved by setting schedulers. NVMe Drives do not really benefit from this however we can create a config that alows us to automatically set NVMe Drives to none and set any SATA drives to run on BFQ we can do this by running

`sudo nano /etc/udev/rules.d/60-ioschedulers.rules`

Once this new file is open just paste the following

```
# set scheduler for NVMe
ACTION=="add|change", KERNEL=="nvme[0-9]*", ATTR{queue/scheduler}="none"
# set scheduler for SSD and eMMC
ACTION=="add|change", KERNEL=="sd[a-z]|mmcblk[0-9]*", ATTR{queue/rotational}=="0", ATTR{queue/scheduler}="bfq"
# set scheduler for rotating disks
ACTION=="add|change", KERNEL=="sd[a-z]", ATTR{queue/rotational}=="1", ATTR{queue/scheduler}="bfq"
```

If you would like to run something like mq-deadline on your NVMe Drives by default then on the second like at the end in quotations where it says “none” replace none with mq-deadline and keep the quotations around it then we can save this file by pressing Control + X then y and enter.

## Network Tweaks <a name="network"></a>

`sudo nano /etc/sysctl.d/99-sysctl.conf`

and then copy and paste this

```
net.core.netdev_max_backlog = 16384
net.core.somaxconn = 8192
net.core.rmem_default = 1048576
net.core.rmem_max = 16777216
net.core.wmem_default = 1048576
net.core.wmem_max = 16777216
net.core.optmem_max = 65536
net.ipv4.tcp_rmem = 4096 1048576 2097152
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.udp_rmem_min = 8192
net.ipv4.udp_wmem_min = 8192
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 2000000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 10
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_keepalive_time = 60
net.ipv4.tcp_keepalive_intvl = 10
net.ipv4.tcp_keepalive_probes = 6
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_timestamps = 0
net.ipv4.ip_local_port_range = 3000 65535
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.log_martians = 1
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.default.secure_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.icmp_echo_ignore_all = 1
net.ipv6.icmp.echo_ignore_all = 1
//Not a network tweak but needed for games like DayZ to load the world and not crash
vm.max_map_count = 1048576
```

Then press Control + X to save then y and enter. This is just a bunch of options that will allow more incoming and outgoing connections as well as opening a majority of the ports of your system which you can change for security reasons or leave open if you game need them. You can read more about these tweaks on the Arch Wiki here - https://wiki.archlinux.org/title/Sysctl#Increasing_the_size_of_the_receive_queue.

## initramfs tweaks <a name="initramfs"></a>

Your going to want to make sure everything we need is properly loading up in the initial boot stages of our system and we can do this with a very simple tool called hwdetect you can install this by running

`sudo pacman -S hwdetect`

and then run

`sudo hwdetect --filesystem`

we are going to want to add the missing MODULES to our mkinitcpio file by running

`sudo nano /etc/mkinitcpio.conf`

after adding those extra modules your also going to want to go down to the next Option called BINARIES are make sure it matches on of the following

<b>BTRFS (Single Device)</b><br>
`BINARIES="fsck fsck.btrfs btrfsck"`

<b>BTRFS (Multi Device)</b><br>
`BINARIES="fsck fsck.btrfs btrfs btrfsck"`

<b>EXT4</b><br>
`BINARIES="fsck fsck.ext[2|3|4] e2fsck"`

<b>XFS</b><br>
`BINARIES="fsck fsck.xfs xfs_repair"`

<b>VFAT (FAT32) Usually Bootloaders</b><br>
`BINARIES="fsck fsck.vfat dosfsck"`

You are more then welcome to add multiple of these like EXT4 and XFS just don’t type in fsck twice as it’s already been loaded and then you can again Control + X and then press y and enter to save the file and exit. Lastly we just need to reconfigure the kernel by running

`sudo mkinitcpio -P`

## Laptop Battery Improvement <a name="laptop"></a>

You can increase you battery life by adding the tlp package and enabling the systemd process like this

```
pacman -S tlp
systemctl enable tlp.service –now
```

This may decrease performance a bit so feel free to remove the package if it slows your system down to much.

## Fusuma Touchpad Gestures <a name="touchpad"></a>

This will imrpove the default gestures and you can get these by running

`yay -S ruby-fusuma libinput ruby`

then we need to our user to the input group by running

`sudo gpasswd -a $USER input`

where \$USER is your username then we can install a gem for fusuma by running

`sudo gem install fusuma`

Then a reboot and it should be running.

### Donations <a name="donations"></a>

https://cointr.ee/dante_robinson
