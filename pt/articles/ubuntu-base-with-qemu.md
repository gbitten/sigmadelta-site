---
area: articles
ref: ubuntu-base-with-qemu
title: Ubuntu Base com QEMU
lang: pt
comments: true
---

Este procedimento mostra como criar uma máquina virtual QEMU<sup>[1]</sup> com o Ubuntu Base<sup>[2]</sup> assim como o root filesystem. Existe um excelente post sobre a instalação do Ubuntu Base no Ask Ubuntu<sup>[3]</sup>. Eu sigo o mesmo caminho, porem adiciono aqui a instalação em uma máquina virtual QEMU. É importante menciosanr que este procedimento funcionará apenas se o host e a máquina virtual tiverem a mesma arquitetura. Além disso, este procedimento foi testado na plataforma x86. Não posso garantir que irá funcionar em outras arquiteturas. 

## Sobre o Ubuntu Base

O **Ubuntu Base** é um rootfs despojado, que permite instalar qualquer pacote do repositório de pacotes do Ubuntu usando o comando `apt-get`. O Ubuntu Base foi anteriormente conhecido como Ubuntu Core, mas a Canonical o renomeou e repassou este nome para outro projeto. 

## Sobre o QEMU

O **QEMU** é um hypervisor ou monitor de máquina virtual de código aberto. Ele pode virtualizar hardware de várias arquiteturas, incluindo x86, SPARC, ARM, PowerPC, MIPs, Microblaze entre outras.

## Download do Ubuntu Base

Escolha a versão do Ubuntu Base [here](http://cdimage.ubuntu.com/ubuntu-base/releases/) e faça o download. Neste exemplo, eu uso a versão 16.04. Se se o host for 32 bits entã a máquina virtual também deve ser de 32 bits. 

Se o host é 32 bits então faça o download do Ubuntu Base 32 bits:

```
wget http://cdimage.ubuntu.com/ubuntu-base/releases/16.04/release/ubuntu-base-16.04-core-i386.tar.gz
```

Se o host é 64 bits então faça o download do Ubuntu Base 64 bits:

```
wget http://cdimage.ubuntu.com/ubuntu-base/releases/16.04/release/ubuntu-base-16.04-core-amd64.tar.gz
```

## Instalar QEMU

```
sudo apt-get install qemu
```

## Criar imagem QEMU

```
qemu-img create -f raw ubuntu-base.img 500M
```

## Logar como root

Todos os comando a seguir requerem privilégio de root.

```
sudo su
```

## Criar um Network Block Device

QEMU pode exportar uma image de disco cmo um Network Block Device (NBD). Como ele, o host pode manipular a imagem da mesma maneria que seria com outro block device.  

```
modprobe nbd max_part=16
qemu-nbd -f raw -c /dev/nbd0 ubuntu-base.img
```

## Partição do device

```
fdisk /dev/nbd0
```

Então precione `n` `p` `1` `<Return>` `<Return>` `a` `1` `w`. Esta sequência cria o device com uma única partição, sendo ela de boot.

## Formatar a partição

```
mkfs.ext4 /dev/nbd0p1
```

## Montar a partição

```
mount /dev/nbd0p1 /mnt
```

## Instalar o Ubuntu Base

Se o host for 32 bits:

```
tar xvfz ubuntu-base-16.04-core-i386.tar.gz -C /mnt
```

Se o host for 32 bits:

```
tar xvfz ubuntu-base-16.04-core-amd64.tar.gz -C /mnt
```

## Copiar configuração de DNS

```
cp /etc/resolv.conf /mnt/etc/resolv.conf
```

## Rodar `chroot`

```
mount --rbind /sys /mnt/sys
mount --rbind /proc /mnt/proc
mount --rbind /dev /mnt/dev
chroot /mnt
```

### Instalar o kernel

```
apt-get update 
apt-get install linux-image-4.4.0-75-generic
```

**ATENÇÃO**: ao final desta instalação se questionado: `GRUB install devices`. Escolha a opção relacionada ao dispositivo nbd0p1. Qualquer outra opção pode danificar o bootloader do host. 

### Ajusta a configuração do grub

```
sed -i "s/quiet splash/rw text/g" /etc/default/grub
update-grub
sed -i "s/nbd0p1 ro/sda1/g" /boot/grub/grub.cfg
```

###  Mudar a senha do root

```
passwd root
```

### Sair do ```chroot```

```
exit
```

## Instalar grub
```
grub-install --root-directory=/mnt /dev/nbd0
```

## Reiniciar o host

```
reboot
```

## Rodar a virtual machine

Se a máquina virtual for 32 bits:

```
qemu-system-i386 ubuntu-base.img
```

Se a máquina virtual for 64 bits:

```
qemu-system-x86_64 ubuntu-base.img
```

## Referências

* [1] [QEMU](http://www.qemu.org/)
* [2] [Ubuntu Base](https://wiki.ubuntu.com/Base)
* [3] [What commands are needed to install Ubuntu Core?](https://askubuntu.com/a/70139/413551)

