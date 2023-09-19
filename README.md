# OpenBSD


## Adicionando um novo pacote ao sistema
Vamos adicionar a ferramenta **sudo** à nossa instalação de OpenBSD. Para isso vamos instalar o pacote da ferramenta:

```ksh
pkg_add sudo

#quirks-3.187 signed on 2020-05-19T14:41:48Z
#Ambiguous: choose package for sudo
#a       0: <None>
#        1: sudo-1.8.31
#        2: sudo-1.8.31-gettext
#        3: sudo-1.8.31-gettext-ldap
Your choice: 1
#sudo-1.8.31: ok
```

### Dando permissões para usuários do grupo wheel usar o programa `sudo`

Verifique o arquivo `/etc/sudoers` com **visudo**

```ksh
visudo
```
Remova o comentário para permitir que as pessoas no grupo wheel executem todos os comandos e defina variáveis de ambiente.

```ksh
# conteúdo do arquivo /etc/sudoers
# descomente essa linha
%wheel  ALL=(ALL) SETENV: ALL
```
Salve e saia do editor *visudo*. Digite ESC, depois :WQ e ENTER
>>> Nota: O utilitário visudo realiza a verificação de sintaxe antes de enviar suas edições no arquivo. Um arquivo sudoers malformado pode danificar seu sistema. Nunca edite /etc/sudoers diretamente. Por exemplo, se você cometer um erro, verá isso ao sair do visudo.

## Adicionar Usuários

```ksh
useradd -m miguel
passwd
Changing password for miguel.
New password: (digite_sua_senha)
Retype new password: (redigite_sua_senha)
```

## Adicionar o Usuário a um grupo
Lembre-se que, para poder executar o comando `su`, seu usuário deve pertencer ao grupo **wheel**. Então vamos adicionar meu usuário ao grupo **wheel**.
```ksh
user mod -G wheel miguel
```



## Modulos do Kernel

Os módulos do kernel NetBSD e OpenBSD terminam com `.o.`
Os módulos do kernel do NetBSD estão em `/usr/lkm`. FAZER: E OpenBSD
NetBSD e OpenBSD usam as ferramentas `modload`, `modunload` e `modstat`.

## Criando partições

Vamos ver como o openbsd trata discos e partições.

### Listando seus discos de armazenamento

Para listar os discos reconhecidos pelo kernel após a inicialização faça 

```ksh
sysctl hw.disknames
# Saída
hw.disknames=wd0:bfb4775bb8397569,cd0:,wd1:56845c8da732ee7b,wd2:f18e359c8fa2522b
```
Os discos são identificados por Disklabel Unique Identifiers (DUIDs) no arquivo fstab(5) por padrão. DUIDs são números aleatórios de 16 dígitos hexadecimais gerados quando um disklabel é criado pela primeira vez.
por exemplo, o disco wd0 é identificado por `bfb4775bb8397569`.

- Se seu disco for SATA você verá `sd0` para o primeiro disco e `sd1` para o segundo e assim por diante
- Se seu disco for  ATA você verá `wd0` para o primeiro disco e `wd1` para o segundo e assim por diante


### Particionando o disco
Supondo que você vá particionar o disco reconhecido como `sd0` (que pode ser um pendrive ou um disco sata)

```ksh
fdisk sd0

## Saída
Disk: sd0       geometry: 553/255/63 [8883945 Sectors]
Offset: 0       Signature: 0xAA55
         Starting       Ending       LBA Info:
 #: id    C   H  S -    C   H  S [       start:      size   ]
------------------------------------------------------------------------
 0: 12    0   1  1 -    2 254 63 [          63:       48132 ] Compaq Diag.
 1: 00    0   0  0 -    0   0  0 [           0:           0 ] unused
 2: 00    0   0  0 -    0   0  0 [           0:           0 ] unused
*3: A6    3   0  1 -  552 254 63 [       48195:     8835750 ] OpenBSD
```

O identificador do tipo da partição BSD (do kernel OpenBSD) deve ser `A6`.

### Criando o layout da partição BSD do disco:
O comando `disklabel` vai funcionar como um editor de layout da partição BSD no disco escolhido. 
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
Para não ficar engessado, você pode obviamente alterar todo o layout de particionamento montando a raiz apenas e recriando a árvore de diretórios dentro dela.

### O esquema de particionamento Unix BSD
- A letra `a` é a partição raiz
- A letra `b` é a partição de swap
- A letra `c` representa todo espaço em disco que você escolheu para particionar
Como estamos lidando com partições no formato BSD 4.2, precisamos alocar todo espaço que vamos usar para fazer o layout das partições.

### Particionando
Vamos editar o layout da partição bsd do disco `wd0`

```ksh
disklabel -E wd0
```
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
Fonte [https://www.openbsdhandbook.com/](https://www.openbsdhandbook.com/)
