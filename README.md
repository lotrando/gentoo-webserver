# Realist's Webserver on Gentoo Linux

## About this project


## Final desktop - One monitor screenshot

<img src="" alt="One Monitor Screenshot" />

## Create install environment

### Partitions
```
parted -s /dev/sda mklabel gpt && parted -a optimal /dev/sda
```

#### Parted commands

```
unit mib
mkpart primary fat32 1 150
name 1 UEFI
set 1 bios_grub on
mkpart primary 150 -1
name 2 ROOT
quit
```

```
mkfs.fat -n UEFI -F32 /dev/sda1 && mkfs.f2fs -l ROOT -O extra_attr,inode_checksum,sb_checksum -f /dev/sda2
```

```
mkdir -p /mnt/gentoo && mount -t f2fs /dev/sda2 /mnt/gentoo
```

```
mkdir -p /mnt/gentoo/boot && mount /dev/sda1 /mnt/gentoo/boot
```

### Stage3 and config portage

```
cd /mnt/gentoo
```

```
wget https://distfiles.gentoo.org/releases/amd64/autobuilds/20240519T170355Z/stage3-amd64-openrc-20240519T170355Z.tar.xz
```

```
tar xpf stage3-amd64-openrc-20240519T170355Z.tar.xz --xattrs-include='*.*' --numeric-owner
```

```
mkdir -p /mnt/gentoo/var/db/repos/gentoo && mkdir -p /mnt/gentoo/etc/portage/repos.conf
```

```
cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/
```

```
cp /etc/resolv.conf /mnt/gentoo/etc/
```

### Mounting important system FS

```
mount -t proc none /mnt/gentoo/proc && mount -t sysfs none /mnt/gentoo/sys
```

```
mount --rbind /sys /mnt/gentoo/sys && mount --make-rslave /mnt/gentoo/sys
```

```
mount --rbind /dev /mnt/gentoo/dev && mount --make-rslave /mnt/gentoo/dev
```

```
mount --rbind /run /mnt/gentoo/run && mount --make-rslave /mnt/gentoo/run
```

```
test -L /dev/shm && rm /dev/shm && mkdir /dev/shm
```

```
mount --types tmpfs --options nosuid,nodev,noexec shm /dev/shm && chmod 1777 /dev/shm
```

### Chroot to prepared system

```
chroot /mnt/gentoo /bin/bash && env-update && source /etc/profile
```

### Sync and config portage

```
emerge-webrsync
```

```
cd /etc/portage/
```

```
rm make.conf && rm -R package.use && rm -R package.accept_keywords && rm -R package.mask
```

### File - /etc/portage/make.conf

```
wget https://raw.githubusercontent.com/lotrando/realist-hyprland-desktop/main/make.conf
```

```
# RXMD - Realist Xmonad Minimal Desktop - make.conf file (c) 2024 -> /etc/portage/make.conf

USE="dbus elogind pipewire nls vulkan wayland"
CPU_FLAGS_X86="aes avx avx2 f16c fma3 mmx mmxext pclmul popcnt rdrand sse sse2 sse3 sse4_1 sse4_2 ssse3"

COMMON_FLAGS="-O2 -pipe -fomit-frame-pointer"
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
FCFLAGS="${COMMON_FLAGS}"
FFLAGS="${COMMON_FLAGS}"
MAKE_OPTS="-j6"

GENTOO_MIRRORS="https://mirror.dkm.cz/gentoo/"
PORTAGE_BINHOST="http://94.113.201.164:55/hyprland"
PORTDIR="/var/db/repos/gentoo"
DISTDIR="/var/cache/distfiles"
PKGDIR="/var/cache/binpkgs"
PORTAGE_NICENESS=19
PORTAGE_IONICE_COMMAND="ionice -c 3 -p \${PID}"
EMERGE_DEFAULT_OPTS="-v --ask-enter-invalid --jobs=1 --load-average=6"
FEATURES="downgrade-backup parallel-fetch sign"

ACCEPT_KEYWORDS="amd64"
ACCEPT_LICENSE="-* @FREE"
GRUB_PLATFORMS="pc efi-64"

LC_ALL=C
LC_MESSAGES=C
L10N="cs"

INPUT_DEVICES="libinput"
VIDEO_CARDS="amdgpu radeonsi"
```

### File - /etc/portage/package.accept_keywords

```
wget https://raw.githubusercontent.com/lotrando/realist-hyprland-desktop/main/package.accept_keywords
```

```
# RHMD - Realist Hyprland Minimal Desktop - package.accept_keywords file -> /etc/portage/package.accept_keywords

# APP-EDITORS
app-editors/sublime-text ~amd64

# APP-MISC
app-misc/nwg-look ~amd64

# DEV-CPP
dev-cpp/sdbus-c++ ~amd64

# GUI-APPS
gui-apps/hypridle ~amd64
gui-apps/hyprlock ~amd64
gui-apps/hyprpaper ~amd64
gui-apps/hyprpicker ~amd64
gui-apps/rofi-wayland ~amd64
gui-apps/waybar ~amd64
gui-apps/wlr-randr ~amd64

# GUI-LIBS
gui-libs/xdg-desktop-portal-hyprland ~amd64

# GUI-WM
gui-wm/hyprland-contrib ~amd64

# APP-MISC ! important
app-misc/ca-certificates ~amd64

# SYS-KERNEL ! important
sys-kernel/zen-sources ~amd64

# APP-SHELLS ! important
app-shells/oh-my-zsh ~amd64
app-shells/zsh-autosuggestions ~amd64
app-shells/zsh-syntax-highlighting ~amd64

# MEDIA-VIDEO ! important
media-video/pipewire ~amd64
media-video/wireplumber ~amd64

# SYS-APSS
sys-apps/eza ~amd64

# X11-APPS - nwg-look
x11-apps/xcur2png ~amd64

# X11-THEMES - nwg-look
x11-themes/elementary-xfce-icon-theme ~amd64
```

### File - /etc/portage/package.use

```
wget https://raw.githubusercontent.com/lotrando/realist-hyprland-desktop/main/package.use
```

```
# RHMD - Realist Hyprland Minimal Desktop - package.use file -> /etc/portage/package.use

# APP-MISC
app-misc/mc nls -slang unicode gpm sftp

# DEV-LIBS
dev-libs/libdbusmenu gtk3

# DEV-QT
dev-qt/qtgui egl

# DEV-VCS
dev-vcs/git -perl

# GUI-APPS
gui-apps/rofi-wayland drun windowmode
gui-apps/waybar pulseaudio udev network tray upower wifi

# GUI-LIBS
gui-libs/wlroots x11-backend X

# GUI-WM
gui-wm/hyprland X

# MEDIA-FONTS
media-fonts/terminus-font -ru-g

# MEDIA-LIBS
media-libs/libsdl2 gles2
media-libs/libglvnd
media-libs/vulkan-loader

# MEDIA-PLUGINS
media-plugins/audacious-plugins nls pipewire cdda cue ffmpeg flac http lame libnotify modplug mp3 opus sndfile wavpack

# MEDIA-SOUND
media-sound/pulseaudio alsa-plugin -bluetooth -daemon

# MEDIA-VIDEO
media-video/pipewire sound-server v4l -bluetooth
media-video/ffmpeg mp3 v4l webp
media-video/mpv cdda dvd jpeg
media-video/ffmpeg bzip2 dav1d encode gnutls gpl iconv mp3 network pic postproc threads v4l vulkan webp zlib modplug opus svg x264 x265 xvid

# SYS-BOOT
sys-boot/grub mount

# SYS-KERNEL
sys-kernel/linux-firmware initramfs
sys-kernel/zen-sources symlink

# X11-LIBS
x11-libs/libdrm video_cards_radeon

#media-libs/freetype -X
#app-misc/mc -X
#media-fonts/liberation-fonts -X
#media-fonts/terminus-font -X
#media-fonts/ubuntu-font-family -X
#media-video/pipewire -X
#media-libs/libpulse -X

dev-cpp/cairomm X
dev-cpp/gtkmm X

x11-libs/cairo X
x11-libs/gtk+ X

dev-qt/qtbase opengl
dev-qt/qttools opengl

app-crypt/pinentry X
sys-apps/dbus X
```

### Edit file - /etc/portage/package.license

```
wget https://raw.githubusercontent.com/lotrando/realist-hyprland-desktop/main/package.license
```

```
# RHMD - Realist Hyprland Minimal Desktop - package.license file -> /etc/portage/package.license

# APP-EDITORS
app-editors/vscode Microsoft-vscode
app-editors/sublime-text Sublime

# SYS-KERNEL
sys-kernel/linux-firmware linux-fw-redistributable no-source-code
```

### Edit file - /etc/portage/package.mask

```
wget https://raw.githubusercontent.com/lotrando/realist-hyprland-desktop/main/package.mask
```

```
# RHMD - Realist Hyperland Minimal Desktop - package.mask file -> /etc/portage/package.mask
```

```
sed -i 's/UTC/local/g' /etc/conf.d/hwclock
```

### Edit file - /etc/fstab

```
nano /etc/fstab
```

## SSD or SATA Disk

```
/dev/sda1         /boot   vfat    noatime       0 2
/dev/sda2         /       f2fs    defaults,rw   0 0
```

```
sed -i 's/localhost/hyprland/g' /etc/conf.d/hostname
```

```
sed -i 's/default8x16/ter-v16b/g' /etc/conf.d/consolefont
```

```
sed -i 's/us/cz/g' /etc/conf.d/keymaps
```

```
sed -i 's/127.0.0.1/#127.0.0.1/g' /etc/hosts
```

```
echo "127.0.0.1 hyprland.gentoo.dev hyprland localhost" >> /etc/hosts
```

### Edit file - /etc/locale.gen

```
nano /etc/locale.gen
```

```
cs_CZ.UTF-8 UTF-8
cs_CZ ISO-8859-2
```

### Edit file - /etc/env.d/02locale

```
nano /etc/env.d/02locale
```

```
LANG="cs_CZ.UTF-8"
LC_COLLATE="C"
```

```
echo "Europe/Prague" > /etc/timezone
```

### Create locale

```
locale-gen
```

```
eselect locale set 7
```

```
env-update && source /etc/profile
```

```
export PS1="(chroot) ${PS1}"
```

### Edit file - /etc/conf.d/net

## Static network (variable, default dhcp)

```
nano /etc/conf.d/net
```

```
config_enp0s3="192.168.0.30 netmask 255.255.255.0"
routes_enp0s3="default via 192.168.0.1"
```

```
cd /etc/init.d/
```

```
ln -s net.lo net.enp0s3
```

## Compiling phase

### Recompile compilers [ enable LTO GPO ] - Not Important
```
emerge gcc python rust clang
```

### Create zen-kernel and install important system packages 
59 packages
```
emerge seatd dhcpcd grub terminus-font sudo f2fs-tools dev-vcs/git eselect-repository genkernel linux-firmware zen-sources --noreplace nano && genkernel all
```

### Create user (replace realist and toor with custom user and password)

```
useradd -m -G audio,video,usb,cdrom,portage,users,wheel,seat -s /bin/bash realist
```

```
echo "root:toor" | chpasswd -c SHA256
```

```
echo "realist:toor" | chpasswd -c SHA256
```

### Add repository overlay mv for OH-MY-ZSH

```
eselect repository enable mv && emaint sync -r mv && emerge --oneshot sys-apps/portage
```
```
emerge oh-my-zsh gentoo-zsh-completions zsh-completions
```

### Install oh-my-zsh plugins and theme

```
git clone https://github.com/romkatv/powerlevel10k.git /usr/share/zsh/site-contrib/oh-my-zsh/custom/themes/powerlevel10k
```
```
git clone https://github.com/zsh-users/zsh-autosuggestions.git /usr/share/zsh/site-contrib/oh-my-zsh/custom/plugins/zsh-autosuggestions
```
```
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git /usr/share/zsh/site-contrib/oh-my-zsh/custom/plugins/zsh-syntax-highlighting
```

## Desktop
```
eselect repository enable guru && emaint sync -r guru
```
```
emerge hyprland hyprpaper hyprpicker hyprland-contrib xdg-desktop-portal-hyprland waybar xwayland
```
```
emerge grim slurp neofetch eix gentoolkit kitty imagemagick ubuntu-font-family gnome-themes-standard elementary-xfce-icon-theme rofi-wayland qt5ct adwaita-qt pipewire firefox-bin mpv audacious nwg-look pulsemixer
```

### Install WEB developers packages ( optional )

```
emerge phpmyadmin dev-db/mysql dev-lang/php nodejs composer vscode sublime-text
```

### Set PHP version for CLI and APACHE

```
eselect php set cli php8.2 && eselect php set apache2 php8.2
```

### Config WEB Develop enviroment PHP, APACHE, MYSQL, PHPMYADMIN

```
rm -R /usr/lib/tmpfiles.d/mysql.conf
```

```
echo "d /run/mysqld 0755 mysql mysql -" > /usr/lib/tmpfiles.d/mysql.conf
```

```
sed -i 's/SSL_DEFAULT_VHOST/PHP/g' /etc/conf.d/apache2
```

```
echo "ServerName localhost" >> /etc/apache2/httpd.conf
```

```
rm -R /var/www/localhost/htdocs/index.html && echo "<?php phpinfo(); ?>" > /var/www/localhost/htdocs/index.php
```

```
cp /var/www/localhost/htdocs/phpmyadmin/config.sample.inc.php /var/www/localhost/htdocs/phpmyadmin/config.inc.php
```

```
mkdir /var/www/localhost/htdocs/phpmyadmin/tmp/
```

```
chown -R apache:apache /var/www/ && usermod -aG apache realist
```

```
chmod -R 775 /var/www/localhost/htdocs && chmod -R 777 /var/www/localhost/htdocs/phpmyadmin/tmp
```

### Add Blowfish secret to phpmyadmin

```
nano /var/www/localhost/htdocs/phpmyadmin/config.inc.php
```

```
$cfg['blowfish_secret'] = 'WntN0150l71sLq/{w4V0:ZXFv7WcB-Qz';
```

### Mysql root password

```
emerge --config mysql
```

## Configurations

### Grubdev-python/pygobject
```
nano /etc/default/grub
```
```
GRUB_GFXMODE=1920x1080x32
GRUB_GFXPAYLOAD_LINUX=keep
GRUB_BACKGROUND="/boot/grub/grub.png"
GRUB_DISABLE_OS_PROBER=0
GRUB_DEFAULT=0
GRUB_TIMEOUT=5
```

### Sudo

```
sed -i 's/# %wheel ALL=(ALL:ALL) ALL/%wheel ALL=(ALL:ALL) ALL/g' /etc/sudoers
```

### USER - dotfiles setting

```
cd /home/realist
```
```
chown -R realist:realist /home/realist/
```

### Change default shell to OH-MY-ZSH

```
chsh -s /bin/zsh root && chsh -s /bin/zsh realist
```

### Grub Install
```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=HYPRLAND --recheck /dev/sda
```
```
cd /boot/grub && wget -q wget https://raw.githubusercontent.com/lotrando/realist-hyprland-desktop/main/grub.png
```
```
grub-mkconfig -o /boot/grub/grub.cfg
```

### Run daemons

```
rc-update add elogind boot && rc-update add consolefont default && rc-update add numlock default
```
```
rc-update add sshd default && rc-update add dbus default && rc-update add alsasound default
```
```
rc-update add dhcpcd default && rc-update add seatd default
```


### Cleaning and reboot to Hyprland desktop

```
cd / && umount -R /mnt/gentoo && reboot
```
