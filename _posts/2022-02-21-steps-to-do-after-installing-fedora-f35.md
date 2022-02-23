---
layout: post
title:  "Steps to do after installing Fedora 35 (F35)"
sub_title: "F35 KDE Spin to be specific"
excerpt_separator:  <!-- more -->
categories:
  - Howto
tags:
  - Fedora
  - Installation
  - Documentation
---

Post installation steps for Fedora F35.

<!-- more -->

_Disclaimer: This is a work in progress._

What I did.

## Basics

- Focus
    - Set Focus Stealing Prevention to High (Window Behavior)
    - Focus follows mouse (delay 300ms)
    - Raise on hover, delayed by 750ms
- Connetivity
    - Enable internet access
    - Attach bluetooth mouse/keyboard
- Update
    - ~~Let everything update using **Discover**~~ (it pops up in the panel and I let it do it's thing - I guess dnf is working below. konsole->top tells me it's packagekitd, hm... I see kwin_wayland already working here, nice...)
    - ~~A restart might make sense now, but I didn't do it~~
- Data
    - Mount all external filesystems (No rsync et al., I just mount it where I need it, backup is done via NAS)
    - I'm enjoying vanilla KWrite to write this md-file documenting my process...
- Display
    - Get the Displays in right order and set zoom to 175%
    - Set the Rendering backend for the Compositor to OpenGL 3.1 (was 2.0)
    - Set Theme to Breeze Twilight with Colors set to Breeze (Similar to my favorite Arc Darker)
- Usability
    - Launcher -> Settings -> Configure enabled search plugins -> Disable: Bookmarks, Browser History and Browser Tabs
    - Activate Night Color
    - Change Wallpaper to something with 🏔s
- Privacy
    - Mute microphones
- Misc
    - Get familiar with Keyboard Shortcuts
        - I really enjoy the 'activities' feature (meta+q)
        - and KRunner (alt+space)
    - Set Digital Clock
        - To 24h
        - ISO Date (YYYY-MM-DD)
    - Move panel to top and shrink to 32px
    - Can't remove system sounds (which seems to be a [bug](https://bugs.kde.org/show_bug.cgi?id=448705))
    - Enable Tap-to-click
    - Enable Invert scroll direction (Natural scrolling)

## Fedora OS

### DNF Flags for faster updates

```bash
echo 'fastestmirror=1' | sudo tee -a /etc/dnf/dnf.conf
echo 'max_parallel_downloads=7' | sudo tee -a /etc/dnf/dnf.conf # You can change it to 10 or other depending on your connection speed
echo 'deltarpm=true' | sudo tee -a /etc/dnf/dnf.conf
cat /etc/dnf/dnf.conf
```

The last command results in

```bash
[main]
gpgcheck=1
installonly_limit=3
clean_requirements_on_remove=True
best=False
skip_if_unavailable=True
fastestmirror=1
max_parallel_downloads=7
deltarpm=true
```

### Update everything once more

To get practice with `dnf` and also to update the given firmware.

```bash
sudo dnf upgrade --refresh # this will determine the fastest host and probably upgrade a lot => reboot afterwards (because stopped after the kernel, I guess)
sudo dnf clean all
sudo fwupdmgr get-devices
sudo fwupdmgr refresh --force
sudo fwupdmgr get-updates
sudo fwupdmgr update
```

It makes sense to reboot again.

Now the most important thing:

```bash
sudo dnf install -y vim
```

### Fix Dell fan issues

Add i8kmon and dell-bios-fan-control from copr:

```bash
sudo dnf copr enable uriesk/i8kutils
sudo dnf update
sudo dnf install i8kutils dell-bios-fan-control
```

### If you have Thinkpad think about tlp

There are counter arguments, though... read that Fedora gains 30mins with default settings compared to tlp. Forgot where I read that, though. So, YMMV.

### Nice fonts

Manually install `https://download.copr.fedorainfracloud.org/results/dawid/better_fonts/fedora-34-x86_64/02077386-archivo-black-fonts/archivo-black-fonts-1.001-1.fc34.noarch.rpm`. Then:

```bash
sudo dnf copr enable dawid/better_fonts
sudo dnf install fontconfig-enhanced-defaults fontconfig-font-replacements
```

There is a bug report open for F35: [https://github.com/silenc3r/fedora-better-fonts/issues/37](https://github.com/silenc3r/fedora-better-fonts/issues/37)

### Speed up boot time

Find services which take the longest:

```bash
systemd-analyze
systemd-analyze blame
systemd-analyze critical-chain
```

Search online about the services you find what they do and rip out all the shit which is not necessary.

These were mine:

```bash
systemctl disable NetworkManager-wait-online.service # No wait time needed (~6.0sec)
systemctl disable fwupd.service # I prefer to do this manually (~2.2sec)
systemctl disable dnf-makecache.service # Caching is overrated (~6.3sec)
systemctl disable dnf-makecache.timer
```

### Optimize btrfs:

Add `ssd,noatime,space_cache,commit=120` to your fstab. This tells btrfs that we have a SSD, prevent frequent disk writes, make caching of a block quicker and increase the time interval in which data is written to the filesystem.

```bash
sudo systemctl daemon-reload
sudo systemctl enable fstrim.timer
```

### Kernel parameters

```bash
cd /usr/sbin
sudo grubby --update-kernel=ALL --args="processor.ignore_ppc=1 nowatchdog mitigations=off"
```

Restart now.

### Additional Software Sources

```bash
sudo dnf install -y https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
sudo dnf install -y https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

sudo dnf upgrade --refresh
sudo dnf groupupdate core
sudo dnf install -y rpmfusion-free-release-tainted
sudo dnf install -y dnf-plugins-core

flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak update # I do not like snap
```

### Nvidia

Fuck you, Nvidia.

Having said that, let's go forward:

Do have something already inside?

```bash
modinfo -F version nvidia
```

Which probably tells you: *Computer says no*.

```bash
sudo dnf install -y akmod-nvidia
sudo dnf install -y xorg-x11-drv-nvidia-cuda # for cuda/nvdec/nvenc support
sudo dnf install -y xorg-x11-drv-nvidia-cuda-libs
sudo dnf install -y vdpauinfo libva-vdpau-driver libva-utils
sudo dnf install -y vulkan
modinfo -F version nvidia
```

Which now should spit out a version number and you should hear your fans going crazy.

**TODO:** Next laptop will come with AMD.


### Take care of apps watching for file changes

Fix some limits of inotify:

```bash
sudo -i
echo 'fs.inotify.max_user_watches = 524288' >> /etc/sysctl.conf
sysctl -p
```

should answer with:

```bash
fs.inotify.max_user_watches = 524288
```

There are [crazy](https://code.visualstudio.com/docs/setup/linux#_visual-studio-code-is-unable-to-watch-for-file-changes-in-this-large-workspace-error-enospc) [things](https://docs.nextcloud.com/desktop/3.4/faq.html) [happening](https://stackoverflow.com/questions/69199553/how-to-change-the-file-system-watcher-limit-in-kubernetes-fs-inotify-max-user-w)...

## Software

### The usual culprits using Flatpak

Fix a few things flatpak:

```bash
sudo dnf install xdg-desktop-portal-wlr
sudo dnf install gnome-settings-daemon
flatpak install flathub org.gtk.Gtk3theme.Materia-dark-compact
```

```bash
flatpak install -y org.gnu.emacs                # yep, the best editor for lips and clojure-*
flatpak install -y com.vscodium.codium          # another best editor w/o telemetry
flatpak install -y md.obsidian.Obsidian         # organize everything
flatpak install -y shotcut                      # sometimes I edit videos
flatpak install -y flatseal                     # sometimes I don't understand access rights of flatpaks
```

In case your fonts look blurry, there are a [ton](https://github.com/flatpak/flatpak/issues/3619) of [problems](https://www.reddit.com/r/gnome/comments/qeodr3/font_in_flatpak_apps_looks_bad_how_to_fix_it/) and [issues](https://github.com/flatpak/flatpak/issues/2861) on that.

My fix was to explicity change the socket for the affected packages:

```bash
sudo flatpak override --nosocket=wayland your.package.Here
```
## Firefox

- Add Plasma Integration via extension: [https://addons.mozilla.org/en-US/firefox/addon/plasma-integration/](https://addons.mozilla.org/en-US/firefox/addon/plasma-integration/)
- Set default Search Engine to DDG
- Disallow Mozilla to
    - install and run studies
    - send technical and interaction data
- Disable Title Bar


### Shell

Either `zsh` or `fish`. `fish` comes with sane defaults, so `fish`:

```bash
sudo dnf install -y fish util-linux-user
chsh -s /usr/bin/fish
```

Add `~/.local/bin` persistently to `fish`. Do the following (in fish):

```bash
mkdir -p /home/$USER/.local/bin
set -Ua fish_user_paths /home/$USER/.local/bin
```

Check if it is on bash, too:

```bash
bash -c 'echo $PATH'
```

### Nextcloud

```bash
sudo dnf install -y nextcloud-client nextcloud-client-dolphin
```

### git

```bash
sudo dnf install -y git git-lfs
git-lfs install
```

### Multimedia

```bash
sudo dnf groupupdate sound-and-video
sudo dnf install -y libdvdcss
sudo dnf install -y gstreamer1-plugins-{bad-\*,good-\*,ugly-\*,base} gstreamer1-libav --exclude=gstreamer1-plugins-bad-free-devel ffmpeg gstreamer-ffmpeg
sudo dnf install -y lame\* --exclude=lame-devel
sudo dnf group upgrade --with-optional Multimedia
sudo dnf install -y gstreamer1-plugins-base gstreamer1-plugins-good gstreamer1-plugins-ugly gstreamer1-plugins-bad-free gstreamer1-plugins-bad-free gstreamer1-plugins-bad-freeworld gstreamer1-plugins-bad-free-extras ffmpeg
```

### MPV

The best media player.

```bash
sudo dnf install -y mpv
```

### Fedy

```bash
sudo dnf copr enable kwizart/fedy
sudo dnf install -y fedy
```

## The other OSes you have

**TODO: WIP**

## Sources

- https://thomas-leister.de/en/repair-fedora-efi-bootloader/
- https://wiki.gentoo.org/wiki/Flatpak#Fixing_jagged_fonts_on_Wayland
- https://fedoraproject.org/wiki/GRUB_2
- https://ask.fedoraproject.org/t/os-prober-results-not-being-added-to-grub-menu/19145/11
- https://copr.fedorainfracloud.org/coprs/uriesk/i8kutils/
- https://sn99.github.io
- https://mutschler.eu/linux/install-guides/fedora-post-install/

## Troubleshooting:

- Linux Mint is so **massivly behind** Fedora F35, that I had immense problems getting `/boot` and `/boot/efi` straight to dual boot it.
