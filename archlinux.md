## this is my note of archlinux install

- ### preparation
      pacman -S arch-install-scripts gparted xfsprogs dosfstools

- ### format & mount
      mkfs.fat -F 32 /dev/efi_system_partition
      mkfs.xfs -f /dev/root_partition
      mount /dev/root_partition /mnt
      mount --mkdir /dev/efi_system_partition /mnt/boot

- ### install software
      pacstrap -K /mnt base base-devel linux-{zen{,-headers},firmware} grub efibootmgr intel-ucode networkmanager nvidia-open-dkms fish vim
      genfstab -U /mnt >> /mnt/etc/fstab

- ### switch to new root
      arch-chroot /mnt fish

    - #### set passward
          passwd

    - #### config
          chsh -s /bin/fish
          ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
          ln -s /bin/vim /bin/vi
          systemctl enable NetworkManager
          cd /etc     
          vi locale.gen # uncommit en_US.UTF-8 UTF-8
          locale-gen 
          echo "LANG=en_US.UTF-8" > locale.conf
          echo archlinux > hostname
          vi mkinitcpio.conf # remove kms from HOOKS
          mkinitcpio -P
          grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB   
          vi default/grub
          grub-mkconfig -o /boot/grub/grub.cfg
          vi pacman.conf # uncommit multilib and paralleldownloads
          visudo
          useradd -mG wheel -s /bin/fish user
          passwd user

    - #### exit
          umount -R /mnt
          reboot
 
    - #### new user
          sudo pacman -S ntfs-3 noto-fonts-{cjk,emoji,extra} firefox fcitx5-{im,chinese-addons,mozc} code rnote bottom scrcpy git clang mold rustup xorg-{server,xinit} plasma dolphin konsole
          cd
          echo "export DESKTOP_SESSION=plasma && exec startplasma-x11" > .xinitrc
          rustup toolchain install nightly
          mkdir .cargo
          vi .cargo/config.toml
          
        add following things
      
          [target.x86_64-unknown-linux-gnu]
          linker = "clang"
          rustflags = ["-C", "link-arg=-fuse-ld=/usr/bin/mold"]

        - #### install paru
              cd /tmp
              git clone https://aur.archlinux.org/paru.git
              cd paru
              makepkg -si

        - #### install aur nekoray
              paru -S nekoray sing-geo{site,ip}

