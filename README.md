# OpenBSD

This FAQ explains how to create additional partitions for OpenBSD. Some experience with OpenBSD is required to succesfully add a partitions. Specific knowledge of file editing is required to configure the VPS to automatically boot the partition.Note: If not mentioned, a command is always followed by an enter. First you will have to create a partition:

## Criando partições

Vamos ver como o openbsd trata discos e partições.

### Listando seus discos de armazenamento

Para listar os discos reconhecidos pelo kernel após o boot faça 

```ksh
sysctl hw.disknames
# Saída
hw.disknames=wd0:bfb4775bb8397569,cd0:,wd1:56845c8da732ee7b,wd2:f18e359c8fa2522b
```
Os discos são identificados por Disklabel Unique Identifiers (DUIDs) no arquivo fstab(5) por padrão. DUIDs são números aleatórios de 16 dígitos hexadecimais gerados quando um disklabel é criado pela primeira vez.
por exemplo, o disco wd0 é identificado por `bfb4775bb8397569`.

- Se seu disco for SATA você verá `sd0` para o primeiro disco e `sd1` para o segundo e assim por diante
- Se seu disco for  ATA você verá `wd0` para o primeiro disco e `wd1` para o segundo e assim por diante

### Particionando o disco:
O comando `disklabel` vai funcionar como uma espécie de cgdisk do linux. 
```ksh
disklabel -E wd0
```
### O layout fixo criado durante a instalação
Por motivos de "segurança", a instalação do OpenBSD gera um espaço particionado fixo no seguinte formato, onde cada partição é identificada na ferramenta `disklabel` por uma letra.

```
seu_disco.a /
seu_disco.b swap
seu_disco.c ## Espaço particionável
seu_disco.l /home
seu_disco.d /tmp
seu_disco.f /usr
seu_disco.g /usr/X11R6
seu_disco.h /usr/local
seu_disco.k /usr/obj
seu_disco.j /usr/src
seu_disco.e /var ffs
```
Para não ficar engessado, você pode alterar obviamente alterar todo o layout de particionamento montando a raiz apenas e recriando a árvore de diretórios dentro dela.

### O esquema de particionamento Unix BSD
- A letra `a` é a partição raiz
- A letra `b` é a partição de swap
- A letra `c` representa todo espaço em disco que você escolheu para particionar
Como estamos lidando com partições no formato BSD 4.2, precisamos alocar todo espaço que vamos usar para fazer o layout das partições.

### Particionando
Comece inserindo novos limites pressionando `b`. Pressione Enter quando você entrar no Setor Inicial e posteriormente enviar `*`. Com este último comando os limites serão definidos para cobrir todo o seu disco.
Liberamos espaço para a nova partição. Envie, 'a' para criar a partição.
Uma letra de unidade será proposta. Você pode aceitar a proposta pressionando Enter ou escolher uma letra de unidade que ainda esteja livre. De qualquer forma, lembre-se desta carta, pois você precisará dela mais tarde. Para este FAQ usaremos 'x'.
Agora você encontrará três configurações padrão. Você terá que aceitar todas as três propostas pressionando Enter três vezes seguidas.
Para finalizar todas as alterações feitas nas etapas anteriores, pressione 'w'. Feche o disklabel com 'q'.
Agora é hora de criar um novo sistema de arquivos em sua partição. Use o seguinte comando 'newfs wd0X'. Substitua X pela letra da unidade que você escolheu na etapa 4.
Posteriormente, o sistema de arquivos deve ser colocado em um mapa. Este mapa primeiro precisa ser criado. Use o comando 'mkdir /transip' para criar um mapa. Você pode substituir transip por qualquer nome que desejar.
Em seguida, você precisa montar o novo sistema de arquivos no sistema de arquivos existente. Use este comando 'mount /dev/wd0x /transip'. Substitua transip pelo nome do mapa que você escolheu na etapa 8.
Coloque o novo sistema de arquivos em '/etc/fstab' para garantir que a partição seja inicializada automaticamente após uma reinicialização. Caso contrário, você precisará executar o comando do passo 9 sempre que seu VPS for reiniciado.



## Rede

```sh
# Configurar o wi-fi

doas ifconfig iwn0 up
doas ifconfig iwn0 nwid coolwifiname wpakey secretpassword

# ifconfig iwn0 -nwid # para desconectar
dhclient iwn0

# Para deixar a configuração permanente /etc/hostname.<device>
# (e.g. /etc/hostname.iwn0):
#
# ```
# nwid coolwifiname wpakey secretpassword
# dhcp
# reinicia a rede
# sh /etc/netstart 
# ```

# Configurar wireguard
cp wireguard_x230.conf /etc/wireguard/wg0.conf
chmod 700 /etc/wireguard
wg-quick up wg0

# permitir encaminhamento ipv6 em /etc/sysctl.conf
sysctl net.inet6.ip6.forwarding=1

# http://sbc.io/hosts/alternates/fakenews-gambling-porn-social/hosts into /etc/hosts

```

## Atualizações de Sistema

```sh

# para atualizar firmware (porque ele não pode ser acoplado a instalação)
doas fw_update

# para atualizar o sistema
doas syspatch

# instalar alguns pacotes vitais
pkg_add vim git fish htop ttyplot ranger ripgrep pdftk ImageMagick ffmpeg gnuwatch
pkg_add wireguard-tools lynx newsboat gomuks transmission syncthing tor httrack rtl-sdr
pkg_add feh i3 xclip rxvt-unicode symbola-ttf iosevka # (iosevka instead of jetbrains-mono)
pkg_add okular keepassxc vlc
pkg_add firefox tor-browser
pkg_add rust python # (python 3)

# inicializar syncthing
rcctl set syncthing user miguel
# update resource limit for daemon in /etc/login.conf:
#   syncthing:\
#    :openfiles-max=16384:\
#    :openfiles-cur=8192:\
#    :tc=daemon:
rcctl enable syncthing

# arrumar problemas de memória do firefox
usermod -G staff miguel # adicionar usuário ao grupo staff (check by `userinfo miguel`)
# #foo cat /etc/login.conf
#   .... lines ....
#   staff:\
#       :datasize-cur=3072M:\
#       :datasize-max=infinity:\
#       :maxproc-max=512:\
#       :maxproc-cur=256:\
#       :ignorenologin:\
#       :requirehome@:\
#       :tc=default:
#   .... lines ....

cp /etc/examples/doas.conf /etc/
# dando permissão ao usuário para executar comandos como root sem senha (poder final):
#   permit nopass miguel as root

# allow users to power off computer:
#   permit nopass :staff as root cmd zzz
#   permit nopass :staff as root cmd ZZZ
#   permit nopass :staff as root cmd reboot args
#   permit nopass :staff as root cmd shutdown args -p now

```


## Ambiente

```sh
# desativar xconsole na inicialização comentando em /etc/X11/xenodm/Xsetup_0
```

## Montagem de pendrive USB

```sh

disklabel sd0 # veja o nome da partição
mount /dev/sd0i /mnt/usb
umount /mnt/usb

```

## Performance

```sh

# hibernar a 5% da bateria restante
# in /etc/rc.conf.local:
#   ampd_flags=-A -Z 5

# otimizações de bateria
doas rcctl -f start apmd
# https://github.com/krzysztofengineer/openbsd/blob/master/tasks/02-power-management.md#set-the-performance-adjustment-mode
doas rcctl enable ampd
apm -L # sets hw.setperf to 0 (sysctl)
# to cut out image pixels 400 width, 30 height
# convert edc/IMG_20211002_154609-gray.jpg -shave 400x30 tmp/400_30_2.jpg

```

## Audio e Vídeo

```sh

# para desligar o apito de sistema
xset b off

# gravação de audio e vídeo são desativadas por padrão

# - man 4 video -> para gravar video:
doas sysctl kern.video.record=1
doas chown $USER /dev/video
ffmpeg -f v4l2 -input_format mjpeg -video_size 1280x720 -i /dev/video ~/video.mkv # 'q' to stop

# - man 4 audio -> para gravar audio:
doas sysctl kern.audio.record=1

# - man 8 mixerctl -> para ativar o microfone:
doas mixerctl record.adc-0:1_mute=off
aucat -o file.wav # CTRL+C to stop
```
