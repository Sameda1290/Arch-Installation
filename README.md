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

```
pacstrap -K /mnt intel-ucode btrfs-progs base base-devel linux-zen linux linux-zen-headers linux-headers linux-firmware linux-api-headers xdg-user-dirs xorg xorg-xinit xorg-appres sysfsutils xorg-xwayland wayland-utils xorg-xauth vim nano p7zip unzip unrar zip udisks2 gvfs-afc gvfs-mtp gvfs-gphoto2 gphoto2 sudo mkinitcpio git wget curl networkmanager openssh mlocate fastfetch inxi zsh noto-fonts ttf-dejavu ttf-dejavu-nerd ttf-roboto ttf-roboto-mono ttf-roboto-mono-nerd terminus-font gnu-free-fonts noto-fonts noto-fonts-emoji noto-fonts-extra ttf-font-awesome ttf-jetbrains-mono ttf-jetbrains-mono-nerd ttf-liberation ttf-liberation-mono-nerd ttf-nerd-fonts-symbols-mono ttf-nerd-fonts-symbols-common ttf-roboto ttf-roboto-mono ttf-roboto-mono-nerd awesome-terminal-fonts ttf-font-awesome otf-font-awesome pipewire pipewire-pulse pipewire-alsa pipewire-audio pipewire-jack lib32-pipewire lib32-pipewire-jack wireplumber alsa-tools alsa-utils alsa-firmware

genfstab -LUp /mnt >> /mnt/etc/fstab
```

Pacstrap komutu sistemi kurar çekirdek olarak linux ve linux-zen çekirdeklerini seçtik.
<br>
genfstab komutu fstab tablosunu oluşturur

## 2. Aşama
### Chroot ortamına geçiş

```
cp /etc/zsh/zshrc /mnt/root/.zshrc
cp /etc/zsh/zprofile /mnt/root/.zprofile
arch-chroot /mnt
```

### Chroot içindeyiz

```
pacman -Sy reflector curl grub efibootmgr dosfstools mtools os-prober efitools

ln -sf /usr/share/zoneinfo/$timezone /etc/localtime

hwclock --systohc

reflector -c "Turkey," -p https -a 3 --sort rate --save /etc/pacman.d/mirrorlist
```
> [!TIP]
> $timezone kısmına bölgenizi yazınız türkiyedeyseniz Europe/Istanbul yazmalısınız

Burada gerekli bazı paketleri kuruyoruz ve reflector'u çalıştırıyoruz

```
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen

locale-gen

echo "LANG=en_US.UTF-8" >> /etc/locale.conf

echo "FONT=ter-v18n" >> /etc/vconsole.conf

echo "KEYMAP=$keyboardlayout" >> /etc/vconsole.conf

echo "$hostname" >> /etc/hostname

echo "127.0.0.1 localhost" >> /etc/hosts

echo "::1       localhost" >> /etc/hosts

echo "127.0.1.1 $hostname.localdomain $hostname" >> /etc/hosts
```

Bu kısımda bölge, dili ve sistem adını ayarladık şimdi ipucuyla önemli olan kısma geçelim.
> [!TIP]
> $hostname kısmına sisteminize vermek istediğiniz ismi verebilirsiniz. <br> Örnek: ArchLinux

> [!IMPORTANT]
> Dili lütfen değiştirmeyin en_US olarak kalmalı ki yedek sistemini ayarladığımızda sorun çıkmasın.

Yeni adımlarla devam edebiliriz.
```
chsh -s /usr/bin/zsh 

passwd root

useradd -m -G wheel $username

chsh -s /usr/bin/zsh $username

passwd $username

cp /root/.zshrc /home/$username/

cp /root/.zprofile /home/$username/
```

Bu kısımda root şifresini, kullanıcımızı ve şifresini oluşturduk şimdi ipucu bölümüne geçelim.
> [!TIP]
> $username kısmına kullanıcı adınızı girmelisiniz


Sırada servisleri ayarlayalım ve grub'ı kuralım
```
systemctl enable NetworkManager

systemctl enable bluetooth

systemctl enable cups.service

systemctl enable sshd

systemctl enable avahi-daemon

systemctl enable reflector.timer

systemctl enable fstrim.timer

systemctl enable firewalld

systemctl enable acpid

grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB --removable

grub-mkconfig -o /boot/grub/grub.cfg
```
Burada ipucu yok devam edelim.

```
sed -i 's/BINARIES=()/BINARIES=(btrfs setfont)/g' /etc/mkinitcpio.conf

mkinitcpio -P
```
Burada bazı modülleri ayarladık


Şimdi kendimize sudo yetkisi verelim
```
EDITOR=nano sudo -E visudo
```
#%wheel ALL=(ALL:ALL) ALL

Buradaki # işaretini kaldırıyoruz.

Şimdi kendimize wheel yetkisini tanımlayalım.
```
usermod -aG wheel $username
```

2. Aşamanın sonu artık nvidia sürücülerini kurmak için 3. Aşamaya geçebiliriz

## 3. Aşama
### 

























