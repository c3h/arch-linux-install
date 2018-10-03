# install-arch-linux
passo a passo da instalação do Arch Linux ;)

## sobre
Escrevi esse tutorial para aqueles que como eu, tiveram/irão ter problemas em instalar o Arch Linux no modo UEFI e claro, uma forma de manter registro e não ter todo o trabalho de procurar "tudo" novamente na '"raça"'.
Para a instalação e criação deste tutorial, tive que realizar a junção de alguns tutoriais diferentes e que se encontram na [bibliografia] (https://github.com/c3h/install-arch-linux#bibliografia).

### conectando na internet 
	`# wifi-menu`

### testando conexão
	`# ping -c3 archlinux.org`
	
### verificando o modo de inicialização está como UEFI
	`# efivar -l`
	*se o comando listar variáveis EFI significa que você iniciou no modo UEFI
 colocar o teclado em ABNT2
	`# loadkeys br-abnt2`
### particionamento de disco
	`# fdisk -l`
	*lista os discos conectados ao computador, identifique o que você irá usar
	*substitua o 'X' para a letra correspondente do seu disco
	-irei usar a seguinte estrutura:
		>/boot = 300mb
		>/swap = 2gb
		>/raiz = restante do disco
	`# cfdisk /dev/sda`
		*caso apareca 'select label type' escolha 'gpt'	
		/para '/boot'
			> 'new' > 300M > 'type' > EFI System
		/para '/swap'
			> 'new' > 2G > 'type' > Linux swap
		/para '/raiz'
			> 'new' > TECLE APENAS ENTER > 'type' > Linux filesystem
		> 'write' > 'yes' > 'quit'
### formatando as partições
	`# mkfs.fat -F32 /dev/sda1`
	`# mkswap /dev/sda2`
	`# mkfs.ext4 /dev/sda3`
	
### montando as partições
	`# mkdir -p /mnt/boot`
	`# mount /dev/sda1 /mnt/boot`
	`# swapon /dev/sda2`
	`# mount /dev/sda3 /mnt`

### escolhendo o espelho de download
	`# pacman -Sy reflector`
	`# reflector --verbose -l 5 --sort rate --save /etc/pacman.d/mirrorlist`
	
### instalando o arch linux
	`# pacstrap /mnt base`

### configurando o FSTAB
	`# genfstab -U -p /mnt >> /mnt/etc/fstab`

### entrnaod no diretório root do sistema
	`# arch-chroot /mnt`
	
### confugurando o hostname
	`# echo SUBSTITUA-ISSO > /etc/hostname`
	
### configure o teclado > KEYMAP
	`# loadkeys br-abnt2`
	`# echo -e 'KEYMAP="br-abnt2.map.gz"' > /etc/vconsole.conf`

### configurando o idioma
	`# sed -i '/pt_BR/,+1 s/^#//' /etc/locale.gen`
	`# locale-gen`
	`# echo LANG=pt_BR.UTF-8 > /etc/locale.conf`
	`# export LANG=pt_BR.UTF-8`
	
### configurando o fuso horário
	`# ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime`
	*substitua o 'Sao_Paulo' para a região do seu fuso horário
	`# hwclock --systohc --utc`

### configurando a conexão
	Ethernet:
		`# systemctl enable dhcpcd`
	Wifi:
		`# pacman -S wpa_supplicant wpa_actiond dialog iw networkmanager`

### configurando repositorio
	`# sed -i '/multilib\]/,+1 s/^#//' /etc/pacman.conf`
	`# pacman -Syu`

### colocar senha para o usuário root
	`# passwd`

### criar usuario
	`# useradd -m -g users -G wheel,storage,power -s /bin/bash COLOQUE-O-NOME-DO-USUARIO`
	Definir senha do usuário:
	`# passwh COLOQUE-O-NOME-DO-USUARIO`
	
### dando permisão administrativas para os usuarios do grupo wheel
	`# sed -i '/%wheel ALL=(ALL) ALL/s/^#//' /etc/sudoers`
	
### atualizando o 'sudo'
	`# pacman -S sudo`

### instalando o bootloader
	`# bootctl --path=/boot install`
	`# nano /boot/loader/loader.conf`
		>coloque o seguinte conteúdo
			timeout 2
			default arch
	`# nano /boot/loader/entries/arch.conf`
		>coloque o seguinte conteúdo
			```
			title Arch Linux
			linux /vmlinuz-linux
			initrd /initramfs-linux.img
			options root=/dev/sda2 rw
			```

### demontando as partições e reiniciando
	`# exit`
	`# umount -R /mnt`
	`# reboot`

### pós instalação
	>realize o login no sistema
	`$ su`
	`# loadkeys br-abnt2`

### conecte na internet
	`# nmtui`

### testando conexão
	`# ping -c3 archlinux.org`
	
### instalando a base-devel do arch
	`# pacman -S base-devel`

### instalando o XORG 
	`# pacman -S xorg-server xorg-xinit xorg-apps mesa ttf-dejavu gvfs-mtp`
	
### drivers gráficos
	*recomendo que leia: https://github.com/Sup3r-Us3r/Arch-Install#instalar-drivers-gr%C3%81ficos
	`# pacman -S xf86-video-intel`
	
### drivers para a placa de som
	`# pacman -S alsa-utils alsa-lib pulseaudio pulseaudio-alsa pavucontrol`
	
### drivers touchpad
	`# pacman -S xf86-input-synaptics`

### instalando algumas fontes
	`# sudo pacman -S ttf-dejavu ttf-bitstream-vera ttf-liberation`

### gerenciador de login
	`# pacman -S gdm`

### interface gráfica
	*recomendo que leia: https://github.com/Sup3r-Us3r/Arch-Install#instalar-ambiente-de-trabalho
	`# pacman -S gnome`
	
### GDM: ativar inicialização junto com o sistema
	`# systemctl enable gdm.service`

### NetworkManager: ativar inicialização junto com o sistema
	`# systemctl enable NetworkManager.service`
	
### fim
	`# reboot`
	
# bibliografia:
	>http://mindbending.org/pt/configuracao-basica-do-arch-linux-sem-dor
	>https://github.com/Sup3r-Us3r/Arch-Install#gerenciadores-de-janelas
	>https://wiki.archlinux.org/index.php/Installation_guide#UEFI.2FGPT
	>https://forum.archlinux-br.org/viewtopic.php?id=4453
	>https://wiki.archlinux.org/index.php/Systemd-boot#Installation
	>https://antoninopraxedes.wordpress.com/2016/01/16/dual-boot-instalacao-arch-linux-com-windows-10/
	>https://wiki.archlinux.org/index.php/Xorg_(Portugu%C3%AAs)
