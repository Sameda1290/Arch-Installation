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
<br>ikinci komutumuz ise zaman dilimini ayarlıyor.
<br>üçüncü komutumuz saati eşitliyor.
<br>dördüncü komutumuz ntp ayarını aktif ediyor.
<br>beşinci komutumuz zamanı tekrar kaydediyor.
<br>altıncı komutumuz durumu çıktı alıyor.

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

```
pacman -Syy
```
Komutunu kullanmamız lazımki depolarımız yenilensin.

### Disklerin bölümlendirilmesi

```
cfdisk $Diskiniz
```
Buradan disk bölümüne giriyoruz

> [!CAUTION]
> Diskinizdeki verilerin hepsi bu aşamada silinecektir.

```
cfdisk /dev/$Diskiniz
```

> [!TIP]
> $Diskiniz kısmına diskinizi yazın NVME SSD kullanıyorsanız bu nvme0n1 olmalı HDD veya SATA SSD kullanıyorsanız bu sda olmalı

Bölümlendirmeler bu şekilde olabilir

1. EFI
   - 3GB önerilir 

2. SWAP
   - Raminiz 4G veya aşağısındaysa Ramx2 Swap örnek 4GB ram için 4x2=8GB Swap
   - Raminiz 8G ila 64G arasındaysa 8G ila 16G Swap yeterli olabilir.
   - Raminiz 64G üstündeyse Swap önerilmez

3. LINUX'un kendi bölümü
   -Kalan tüm bölümü bununla formatlayınız.

### Disklerin yapılandırılması

Ben, burada btrfs sistem kurulumunu göstereceğim bu yüzden komutlar birazda olsa değişebilir.

```
mkfs.fat -F 32 /dev/$Diskiniz1.bölümü

mkswap /dev/$Diskiniz2.bölümü

swapon /dev/$Diskiniz2.bölümü

mkfs.btrfs -f /dev/$Diskiniz3.bölümü

mount /dev/$Diskiniz3.bölümü /mnt

btrfs su cr /mnt/@

btrfs su cr /mnt/@cache

btrfs su cr /mnt/@home

btrfs su cr /mnt/@snapshots

btrfs su cr /mnt/@log

umount /mnt
```
Yukarıda diskleri formatlayıp subvolume oluşturduk.

Aşağıdaki komutlarda ise subvolumeları yapılandırarak mount ediyoruz
```
mount -o compress=zstd:1,noatime,subvol=@ /dev/$Diskiniz3.Bölümü /mnt

mkdir -p /mnt/{boot/efi,home,.snapshots,var/{cache,log}}

mount -o compress=zstd:1,noatime,subvol=@cache /dev/$Diskiniz3.Bölümü /mnt/var/cache

mount -o compress=zstd:1,noatime,subvol=@home /dev/$Diskiniz3.Bölümü /mnt/home

mount -o compress=zstd:1,noatime,subvol=@log /dev/$Diskiniz3.Bölümü /mnt/var/log

mount -o compress=zstd:1,noatime,subvol=@snapshots /dev/$Diskiniz3.Bölümü /mnt/.snapshots

mount /dev/$Diskiniz1.Bölümü /mnt/boot/efi
```

Aşağıda verdiğim komutla subvolumeların düzgünce mount edilip edilmediğine bakabilirsiniz.

```
btrfs su l /mnt

lsblk -f
```

### Sistemin kurulumu




