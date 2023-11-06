# test

## this is my note of archlinux install

### network connecting
    iwctl station device connect ssid

### format & mount
    mkfs.fat -F 32 /dev/efi_system_partition
    mkfs.xfs -f /dev/root_partition
    mount /dev/root_partition /mnt
    mount --mkdir /dev/efi_system_partition /mnt/boot

### config pacman
    vim /etc/pacman.d/mirrorlist 
    vim /etc/pacman.conf #ParallelDownloads

### install software
    pacstrap -K /mnt base linux-{zen{,-headers},firmware} ntfs-3g grub efibootmgr intel-ucode networkmanager nvidia-open-dkms fish vim noto-fonts-{cjk,emoji,extra} tmux gnome-shell alacritty firefox fcitx5-{im,chinese-addons} code base-devel git clang mold rustup

    genfstab -U /mnt >> /mnt/etc/fstab

### switch to new root
    arch-chroot /mnt fish

### set fish as default shell
    chsh -s /bin/fish

### set passward
    passwd

### set localtime
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

    ln -s /bin/vim /bin/vi

### config
    cd /etc

### set language
    vi locale.gen #en_US.UTF-8 UTF-8
    locale-gen 
    echo "LANG=en_US.UTF-8" > locale.conf

### set hostname
    vi hostname

### for nvidia settings
    vi mkinitcpio.conf #HOOKS kms
    mkinitcpio -P

### config grub
    grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
    vi default/grub #timeout
    grub-mkconfig -o /boot/grub/grub.cfg

### add user
    visudo
    useradd -mG wheel -s /bin/fish username
    passwd username

### config pacman
    vi pacman.conf #multilib paralleldownloads

### enable networkmanager
    systemctl enable NetworkManager

### config user
    su username
    cd

### install cargo
    rustup toolchain install nightly
    mkdir .cargo
    vi .cargo/config.toml

    [target.x86_64-unknown-linux-gnu]
    linker = "clang"
    rustflags = ["-C", "link-arg=-fuse-ld=/usr/bin/mold"]

### config alacritty
    mkdir -p .config/alacritty/themes
    git clone https://github.com/alacritty/alacritty-theme .config/alacritty/themes
    vi .config/alacritty/alacritty.yml

    import:
     - .config/alacritty/themes/themes/tokyo-night-storm.yaml
    font:
     size: 15

### startx
    echo "XDG_SESSION_TYPE=wayland dbus-run-session gnome-session" > startx.sh
    chmod +x ./startx.sh

### install paru
    cd /tmp
    git clone https://aur.archlinux.org/paru.git
    cd paru
    makepkg -si

### install software
    paru -S nekoray sing-geo{site,ip} nvidia-exec opentabletdriver

### install code extensions
    code --install-extension llvm-vs-code-extensions.vscode-clangd
    code --install-extension rust-lang.rust-analyzer
    code --install-extension albert.TabOut
    code --install-extension maziac.asm-code-lens
    code --install-extension enkia.tokyo-night

### git
    git config --global user.email "you@example.com"
    git config --global user.name "Your Name"

### exit
    umount -R /mnt
    reboot

### user login

### config nekoray
    sudo nekoray -many
