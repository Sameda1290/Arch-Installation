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

timedatectl status && date

```

ilk komutumuz ipmizden zaman dilimini çekiyor.
ikinci komutumuz ise zaman dilimini ayarlıyor.
üçüncü komutumuz saati eşitliyor.
dördüncü komutumuz ntp ayarını aktif ediyor.
beşinci komutumuz zamanı tekrar kaydediyor.
altıncı komutumuz durumu çıktı alıyor.

### Pacman'ı ayarlamak

Pacman'ı ayarlamak önemli hem hızlı kurulum hemde bazı paketlerin multilib deposunu gerektirdiği için.

```
nano /etc/pacman.conf
```

Buradan Multilib reposunu gösteren yeri bulup daha sonra # işaretlerini kaldırıp kaydedip çıkıyoruz.

> [!IMPORTANT]
> Testing diye gösterilen repoları açmayınız.

> [!TIP]
> Kaydedip çıkmak için CTRL+X kombosunu yapıp Y tuşuna basıp ENTER tuşuna basıyoruz.





























