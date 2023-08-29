# OpenBSD

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
