# test

## this is my note of archlinux install

- ### network connecting
        iwctl station device connect ssid

- ### format & mount
        mkfs.fat -F 32 /dev/efi_system_partition
        mkfs.xfs -f /dev/root_partition
        mount /dev/root_partition /mnt
        mount --mkdir /dev/efi_system_partition /mnt/boot

- ### config pacman
        vim /etc/pacman.d/mirrorlist 
        vim /etc/pacman.conf

- ### install software
        pacstrap -K /mnt base linux-{zen{,-headers},firmware} ntfs-3g grub efibootmgr intel-ucode networkmanager nvidia-open-dkms fish vim noto-fonts-{cjk,emoji,extra} tmux gnome-{shell,calculator} alacritty firefox fcitx5-{im,chinese-addons,mozc} code rnote bottom base-devel git clang mold rustup

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

        - #### config alacritty
                mkdir -p .config/alacritty/themes
                git clone https://github.com/alacritty/alacritty-theme .config/alacritty/themes
                vi .config/alacritty/alacritty.yml
          
            add following things
          
              import:
                - .config/alacritty/themes/themes/tokyo-night-storm.yaml
              font:
                size: 12

        - #### edit startx
              vi startx.sh
          
          add following things
          
              XDG_SESSION_TYPE=wayland dbus-run-session gnome-session
              sudo nekoray -many
              otd-daemon
     
          give executable permission
          
              chmod +x ./startx.sh

        - #### install paru
              cd /tmp
              git clone https://aur.archlinux.org/paru.git
              cd paru
              makepkg -si

        - #### install aur software
              paru -S nekoray sing-geo{site,ip} opentabletdriver

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
