# test

## this is my note of archlinux install

- ### preparation
  https://github.com/ventoy/Ventoy/releases
  
  https://github.com/MatsuriDayo/nekoray/releases

- ### network connecting
        iwctl station device connect ssid

- ### format & mount
        mkfs.fat -F 32 /dev/efi_system_partition
        mkfs.xfs -f /dev/root_partition
        mkswap /dev/swap_partition
        mount /dev/root_partition /mnt
        mount --mkdir /dev/efi_system_partition /mnt/boot
        swapon /dev/swap_partition

- ### config pacman
        vim /etc/pacman.d/mirrorlist 
        vim /etc/pacman.conf

- ### install software
        pacstrap -K /mnt base linux-{zen{,-headers},firmware} ntfs-3g grub efibootmgr intel-ucode base-devel networkmanager nvidia-dkms fish vim noto-fonts-{cjk,emoji,extra} tmux xorg-{server,xinit} plasma-{desktop,pa} pipewire-{jack,alsa,audio} konsole dolphin firefox fcitx5-{im,chinese-addons,mozc} code rnote bottom scrcpy  git clang mold rustup

- ### genfstab
        genfstab -U /mnt >> /mnt/etc/fstab

- ### switch to new root
        arch-chroot /mnt fish

    - #### set fish as default shell
            chsh -s /bin/fish

    - #### set passward
            passwd

    - #### set localtime
            ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

    - #### create link to vim
            ln -s /bin/vim /bin/vi

    - #### enable system service
            systemctl enable NetworkManager
            systemctl enable systemd-timesyncd.service

    - #### config /etc
            cd /etc

        - #### set language
          edit locale.gen and uncommit `en_US.UTF-8 UTF-8`
          
                vi locale.gen
                locale-gen 
                echo "LANG=en_US.UTF-8" > locale.conf

        - #### set hostname
                echo archlinux > hostname

        - #### for nvidia settings
          edit mkinitcpio.conf and remove `kms` from `HOOKS`
          
                vi mkinitcpio.conf 
                mkinitcpio -P

        - #### install and config grub
                grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB   
          edit grub and add `nvidia_drm.modeset=1 nvidia_drm.fbdev=1` to `GRUB_CMDLINE_LINUX_DEFAULT` then mkconfig
          
                vi default/grub
                grub-mkconfig -o /boot/grub/grub.cfg
          
        - #### config pacman
          uncommit `multilib` and `paralleldownloads`
      
                vi pacman.conf

    - #### add and config user
          visudo
          useradd -mG wheel -s /bin/fish username
          passwd username
          su username
          cd
          echo "export DESKTOP_SESSION=plasma && exec startplasma-x11" > .xinitrc

        - #### install cargo
                rustup toolchain install nightly
                mkdir .cargo
                vi .cargo/config.toml
          
            add following things
      
                [target.x86_64-unknown-linux-gnu]
                linker = "clang"
                rustflags = ["-C", "link-arg=-fuse-ld=/bin/mold"]

        - #### install paru
              cd /tmp
              git clone https://aur.archlinux.org/paru.git
              cd paru
              makepkg -si

        - #### config software
              sudo rmmod wacom

        - #### install code extensions
              code --install-extension llvm-vs-code-extensions.vscode-clangd
              code --install-extension rust-lang.rust-analyzer
              code --install-extension albert.TabOut
              code --install-extension maziac.asm-code-lens
              code --install-extension enkia.tokyo-night

        - #### git
              git config --global user.email "you@example.com"
              git config --global user.name "Your Name"

    - #### exit
          umount -R /mnt
          reboot
