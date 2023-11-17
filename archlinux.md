## this is my note of archlinux install

- ### preparation
        pacman -S archlinux-install-scripts gparted xfsprogs dosfstools

- ### format & mount
        mkfs.fat -F 32 /dev/efi_system_partition
        mkfs.xfs -f /dev/root_partition
        mount /dev/root_partition /mnt
        mount --mkdir /dev/efi_system_partition /mnt/boot

- ### install software
        pacstrap -K /mnt base linux-{zen{,-headers},firmware} ntfs-3g grub efibootmgr intel-ucode networkmanager nvidia-open-dkms fish gvim base-devel git clang mold rustup xorg-{server,xinit} xfce4
  
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
            ln -s /bin/gvim /bin/vi

    - #### enable networkmanager
            systemctl enable NetworkManager

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

        - #### install cargo
                rustup toolchain install nightly
                mkdir .cargo
                vi .cargo/config.toml
          
            add following things
      
                [target.x86_64-unknown-linux-gnu]
                linker = "clang"
                rustflags = ["-C", "link-arg=-fuse-ld=/usr/bin/mold"]

        - #### edit x11
              echo "exec startxfce4" > .xinitrc
          
        - #### install paru
              cd /tmp
              git clone https://aur.archlinux.org/paru.git
              cd paru
              makepkg -si

        - #### install aur nekoray
              paru -S nekoray sing-geo{site,ip}

    - #### exit
          umount -R /mnt
          reboot
