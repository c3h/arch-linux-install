# install-arch-linux
passo a passo da instalação do Arch Linux ;)

## indece
- **[sobre](https://github.com/c3h/arch-linux-install#sobre)**
- **[pré instalação](https://github.com/c3h/arch-linux-install#pr%C3%A9-instala%C3%A7%C3%A3o-conex%C3%A3o-e-disco)**
- **[instalação](https://github.com/c3h/arch-linux-install#instala%C3%A7%C3%A3o)**
- **[configurando sistema](https://github.com/c3h/arch-linux-install#configurando-sistema)**
- **[pós instalação](https://github.com/c3h/arch-linux-install#p%C3%B3s-instala%C3%A7%C3%A3o)**
- **[LEIA TAMBEM](https://github.com/c3h/arch-linux-install/wiki)**

## sobre
Este guia destina-se a ajudar aqueles que como eu, tiveram problemas com a instalação do Arch Linux no modo UEFI e claro, uma forma de manter registro e não ter todo o trabalho de procurar "tudo" novamente na "raça". 

Pressuponho que você tenha alguma familiaridade com o sistema Linux. 

Para a instalação e criação deste tutorial, tive que realizar a junção de alguns tutoriais diferentes e que se encontram na [bibliografia](https://github.com/c3h/install-arch-linux#bibliografia).

A ordem aqui apresentada deve ser seguida para evitar possiveis problemas.

## pré instalação
### conexão
conectando na internet
```
# wifi-menu
```
testando conexão
```
# ping -c3 archlinux.org
```
### verificando se está habilitado o UEFI
```
# efivar -l
```
>se o comando listar variáveis EFI significa que você iniciou no modo UEFI 

colocar o teclado em ABNT2
```
# loadkeys br-abnt2
```
### disco
identifique o disco que será utilizado utilizando o [fdisk](https://wiki.archlinux.org/index.php/Fdisk)
```
# fdisk -l
```
>lista os discos conectados ao computador, identifique o que você irá usar 
>e substitua o 'X' para a letra correspondente do seu disco

irei usar a seguinte estrutura:
- /boot = 500mb
- /swap = 3gb
- /raiz = restante do disco
```
# cfdisk /dev/sdX
```
>caso apareca 'select label type' escolha 'gpt'

para: '/boot'
- 'new' > 500M > 'type' > EFI System

para: '/swap'
- 'new' > 3G > 'type' > Linux swap

para: '/raiz'
- 'new' > TECLE APENAS ENTER > 'type' > Linux filesystem 
- 'write' > 'yes' > 'quit'

formatando as partições
```
# mkswap -L  swap /dev/sda2
# mkfs.ext4 /dev/sda3
# mkfs.fat -F32 -n BOOT /dev/sda1
```
montando as partições
```
# mount /dev/sda3 /mnt
# swapon /dev/sda2
# mkdir -p /mnt/boot
# mount /dev/sda1 /mnt/boot
```
## instalação
escolhendo o espelho de download
```
# pacman -Sy reflector
# reflector --verbose -l 5 --sort rate --save /etc/pacman.d/mirrorlist
```
instalando o arch linux
```
# pacstrap /mnt base
```
## configurando sistema
### configurando o FSTAB
```
# genfstab -U /mnt >> /mnt/etc/fstab
```
>mais informações [aqui](https://pt.wikipedia.org/wiki/Fstab) e [aqui](https://wiki.archlinux.org/index.php/Fstab)

entrando no diretório root do sistema
```
# arch-chroot /mnt
```
configurando o hostname
```
# echo SUBSTITUA-ISSO > /etc/hostname
```
### idioma e localização
configure o teclado > KEYMAP
```
# loadkeys br-abnt2
# echo -e 'KEYMAP="br-abnt2.map.gz"' > /etc/vconsole.conf
```
configurando o idioma
```
# sed -i '/pt_BR/,+1 s/^#//' /etc/locale.gen
# locale-gen
# echo LANG=pt_BR.UTF-8 > /etc/locale.conf
# export LANG=pt_BR.UTF-8
```
>mais informações [aqui](https://wiki.archlinux.org/index.php/System_time)

configurando o fuso horário
```
# ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime
```
>substitua o 'Sao_Paulo' para a região do seu fuso horário
```
# hwclock --systohc --utc
```
### conexão
habilitando ethernet:
```
# systemctl enable dhcpcd
```
### configurando mkinitcpio

```
# mkinitcpio -p linux
```
instalando wifi:
```
# pacman -S wpa_supplicant wpa_actiond dialog iw networkmanager
```
>mais informações [aqui](https://wiki.archlinux.org/index.php/NetworkManager_%28Portugu%C3%AAs%29#Introdu.C3.A7.C3.A3o)
### configurando repositório
```
# sed -i '/multilib\]/,+1 s/^#//' /etc/pacman.conf
# pacman -Syu
```
### instalando o 'sudo'
```
# pacman -S sudo
```
### usuário
colocar senha para o usuário root
```
# passwd
```
criar e configurar usuário
```
# useradd -m -g users -G wheel,storage,power -s /bin/bash COLOQUE-O-NOME-DO-USUARIO
```
definir senha do usuário
```
# passwh COLOQUE-O-NOME-DO-USUARIO
```
dando permisão administrativas para os usuarios do grupo wheel
```
# sed -i '/%wheel ALL=(ALL) ALL/s/^#//' /etc/sudoers
```
### instalando o bootloader
```
# bootctl --path=/boot install
```
- coloque o seguinte conteúdo em:
```
# nano /boot/loader/loader.conf
```
```
timeout 2
default arch
```
- coloque o seguinte conteúdo em:
```
# nano /boot/loader/entries/arch.conf
```
```
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
options root=/dev/sda3 rw
```
>mais informações [aqui](https://wiki.archlinux.org/index.php/Systemd-boot)
### desmontando as partições e reiniciando
```
# exit
# umount -R /mnt
# reboot
```
## pós instalação
>realize o login no sistema
```
$ su
# loadkeys br-abnt2
```
conecte na internet
```
# nmtui
```
testando conexão
```
# ping -c3 archlinux.org
```
instalando base-devel do arch
```
# pacman -S base-devel
```
instalando o XORG
```
# pacman -S xorg-server xorg-xinit xorg-apps mesa ttf-dejavu gvfs-mtp
```
drivers gráficos
>recomendo que leia: [instalar-drivers](https://github.com/Sup3r-Us3r/Arch-Install#instalar-drivers-gr%C3%81ficos)
```
# pacman -S xf86-video-intel
```
drivers para a placa de som
```
# pacman -S alsa-utils alsa-lib pulseaudio pulseaudio-alsa pavucontrol
```
drivers touchpad
```
# pacman -S xf86-input-synaptics
```
instalando algumas fontes
```
# pacman -S ttf-dejavu ttf-bitstream-vera ttf-liberation
```
interface gráfica
>recomendo que leia: [instalar-ambiente-de-trabalho](https://github.com/Sup3r-Us3r/Arch-Install#instalar-ambiente-de-trabalho)
```
# pacman -S gnome
```
>para mais informações sobre o [gnome](https://wiki.archlinux.org/index.php/GNOME)
### GDM: ativar inicialização junto com o sistema
```
# systemctl enable gdm.service
```
### NetworkManager: ativar inicialização junto com o sistema
```
# systemctl enable NetworkManager.service
```
## fim
```
# reboot
```
## bibliografia:
http://mindbending.org/pt/configuracao-basica-do-arch-linux-sem-dor

https://github.com/Sup3r-Us3r/Arch-Install#gerenciadores-de-janelas

https://wiki.archlinux.org/index.php/Installation_guide#UEFI.2FGPT

https://forum.archlinux-br.org/viewtopic.php?id=4453 [RECOMENDO LER]

https://wiki.archlinux.org/index.php/Systemd-boot#Installation

https://antoninopraxedes.wordpress.com/2016/01/16/dual-boot-instalacao-arch-linux-com-windows-10/

https://wiki.archlinux.org/index.php/Xorg_(Portugu%C3%AAs)

https://www.tecmint.com/arch-linux-installation-and-configuration-guide/
