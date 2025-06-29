# Arch-Kurulum Anlatımı  (Turkish ver.)
## 1. Aşama
> [!CAUTION]
> İntel kurulum içindir

> [!CAUTION]
> Disk şifreleme içermez

Arch isosunu hazırlayıp bilgisayar'ı usb den başlatarak 1. aşamaya başlıyoruz.

### Saat dilimini ayarlama

Önce saat dilimini ayarlamak önemli ki sorun yaşamayalım.

```
timezone=$(curl -s https://ipinfo.io/timezone)

ln -sf /usr/share/zoneinfo/$timezone /etc/localtime

hwclock -uw

timedatectl set-ntp 1

timedatectl set-timezone $timezone

timedatectl status && clear && sleep 0.1 && sleep 1 && date

```


