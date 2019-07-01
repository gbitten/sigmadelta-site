---
area: articles
ref: compiling-preempt-rt
title: Compilando o Preempt RT
lang: pt
comments: true
---

Eu costumo a a executar esse procedimento (com algumas variações quanto necessário) para compilar e instalar o kernel do Linux com Preempt RT<sup>[1]</sup> nos meus equipamento com Ubuntu.

## Preparar ambiente

Instalar o toolchain para o build do kernel. 

```
sudo apt-get install git
sudo apt-get install libssl-dev
sudo apt-get install dpkg-devi
sudo apt-get build-dep linux-image-$(uname -r)
```

## Criar diretório para o código do kernel

```
mkdir preempt-rt
cd preempt-rt
```

## Download o kernel do Linux e o conjunto de patches do Preempt RT

Os médotos abaixo fazem praticamente a mesma coisa. A diferença é que o primeiro aplica conjunto de patches de uma vez só, com apenas um commit. Na segunda, o Preempt RT é uma sequencia de commits menores, por tanto é mais fácil entender cada um deles.

### Método 1

Download do kernel mainline.
 
```
git init
git remote add origin git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git
git fetch origin v4.9.18
git reset --hard FETCH_HEAD
```

Aplicar o conjunto de patches do Preempt RT patch.

```
wget -O - https://www.kernel.org/pub/linux/kernel/projects/rt/4.9/older/patch-4.9.18-rt14.patch.gz | gunzip -c > /tmp/patch-4.9.18-rt14.patch
patch -p1 < /tmp/patch-4.9.18-rt14.patch
git add -A .
git commit -m "Apply Preempt RT patch set v4.9.18-rt14"
```

### Método 2

Pegar o Preempt RT do seu repositorio git ([stable](https://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-stable-rt.git) ou [development](https://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-rt-devel.git)).

```
git init
git remote add origin git://git.kernel.org/pub/scm/linux/kernel/git/rt/linux-rt-devel.git
git fetch origin v4.9.18-rt14-rebase
git reset --hard FETCH_HEAD
```

## Configurar o built do kernel

A configuração do built do kernel é definida no arquivo ```.config```.

### Inicializar a configuração do built do kernel

Existem diversas maneiras de inicializar o ```.config```. Nesta, eu uso a configuração do kernel do Ubuntu para inicalizar a configuração do kernel que eu irei compilar.

```
cp /boot/config-$(uname -r) .config
make olddefconfig
```

### Definir configurações do build para sistemas de  tempo real

Use o seu método prefirido de configuração do kernel (ex: ```make menuconfig```) para definir o seguintes parâmetros:

* PREEMPT_RT_FULL = y
* HIGH_RES_TIMERS = y

## Compilar e instalar o kernel

Estes são os principais métodos que uso para compilar e instalar o kernel. 

### Método 1

Método tradicional que instala o kernel na máquina local.

```
make -j $(nproc) LOCALVERSION=-rt14
sudo make modules_install install
```

### Método 2

Este método cria pacotes que pode ser instalados em outras máquinas. O ```make``` do kernel possui diferentes opções de tipos de pacotes<sup>[2]</sup>. Neste, eu usei a opção ```deb-pkg``` que gera pacotes Debian.

```
make -j $(nproc) deb-pkg LOCALVERSION=-rt14
```

O comando acima cria 5 pacotes:

* linux-firmware-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
* linux-image-4.9.18-rt14-rt14-dbg_4.9.18-rt14-rt14-5_amd64.deb
* linux-headers-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
* linux-libc-dev_4.9.18-rt14-rt14-5_amd64.deb
* linux-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb

Instalar o pacotes do kernel.

```
sudo dpkg -i linux-firmware-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
sudo dpkg -i linux-headers-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
sudo dpkg -i linux-image-4.9.18-rt14-rt14_4.9.18-rt14-rt14-5_amd64.deb
```

## Reiniciar e verificar

Ao final da instalação, a configuração do grub será atualizada para incluir a opção do novo kernel. Reinicalize com o novo kernel e o verifique com o comando ```uname -a```.

## Referências

* [1] [Preempt RT patch](https://rt.wiki.kernel.org/index.php/CONFIG_PREEMPT_RT_Patch)
* [2] [Output of kernel's "make help"](https://www.kernel.org/doc/makehelp.txt)
