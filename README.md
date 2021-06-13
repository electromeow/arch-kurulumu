(Arch Linux _(inşa tarihi)_ 2021.06.01 ile test edildi.)

# Arch Linux Kurulum Rehberi (UEFI & BIOS)

Herkese merhaba, bu rehber minimal bir şekilde tamamen sıfırdan sıfırdan sağlıklı şekilde Arch Linux kurulumunu anlatmaktadır.

## Başlayalım

- **[Arch Linux indirme sayfasından](https://www.archlinux.org/download/)** Arch Linux'u indirip bir belleğe yazdıralım.
- Yazdırma bittikten sonra Arch Linux'u başlatalım. Ardından yapacağımız şeyler:

### Türkçe Klavye Düzenini Yükle

```bash
loadkeys trq
```

### İnternet Bağlantısını Kontrol Et

```bash
ping -c 5 google.com
```

- Eğer internet kablosu ile bağlandıysanız zaten internet bağlantınız olacaktır.
- Eğer Wi-Fi kullanıyorsanız `wifi-menu` komutu ile Wi-Fi ağınıza bağlanın.
- Eğer internete iki şekilde de bağlanamıyorsanız telefonunuzu bilgisayara bağlayıp USB bağlantı ayarlarından `Ağ Paylaşma` seçeneğini seçerek bağlanabilirsiniz.
- Eğer bağlandıysanız devam edelim.

### Sistem Tarihini Güncelle

```bash
timedatectl set-ntp true
```

<hr />

## Diski Kurulum İçin Ayarlama

> :warning: Disklerinizi yönetirken son derece dikkatli olun, verileriniz silinirse beni suçlamayın.

## UEFI Sistem İçin

### Disk Bölümleme (UEFI için)

`gdisk` aracı ile diskimizde `EFI Boot ve Root` olmak üzere 2 bölüm açacağız.

- Eğer diskiniz yeni ise ve bölümleme tablosu bulunamadıysa, `g` tuşuna basarak GPT Bölüm tablosu oluşturun.

```bash
gdisk /dev/[disk ismi]
```

- [disk ismi] = bölümlenecek disk, `lsblk` ile diskinizi bulabilirsiniz.

```
n = Yeni bölüm
enter tuşuna bas = ilk bölüm
enter tuşuna bas = ilk sektör olarak
+512M = son sektör olarak (Boot bölüm boyutu)
ef00 = EFI bölüm tipi

n = Tekrardan yeni bölüm
enter tuşuna bas = 2. bölüm
enter tuşuna bas = ilk sektör olarak
enter tuşuna bas = son sektör olarak [Root bölüm boyutu (kalan diski kullanarak)]
8300 ya da enter tuşuna bas = EXT4 Root bölüm tipi

w = kaydet ve çık
```

### Bölümleri Formatlama (UEFI için)

```
mkfs.fat -F32 /dev/[efi bölüm adı]
mkfs.ext4 /dev/[root bölüm adı]
```

### Bölümleri Bağlama (UEFI için)

```
mount /dev/[root bölüm adı] /mnt
mkdir /mnt/boot/efi
mount /dev/[efi bölüm adı] /mnt/boot/efi
```

## MBR (BIOS) Sistem İçin

### Disk Bölümleme (MBR için)

`cfdisk` aracı ile diskimizde `Swap ve Root` olmak üzere iki bölüm oluşturacağız.

- Eğer diskiniz yeni ise ve bölümleme tablosu bulunamadıysa, `msdos` seçeneğini seçerek MSDos Bölümleme Tablosu oluşturun.

```bash
cfdisk /dev/[disk ismi]
```

- [disk ismi] = bölümlenecek disk, `lsblk` ile diskinizi bulabilirsiniz.
- Swap bölümü RAM miktarınızın iki katı olmalı. 16GB ve üstü RAM'iniz varsa gerek yok.

### Bölümleri Formatlama, SWAP Oluşturup Root'u Bağlama (MBR)

#### ROOT Bölümünü EXT4 Olarak Formatlama

```bash
mkfs.ext4 /dev/[root bölüm adı]
```

#### Swap Bölümünü Ayarlayıp Aktifleştir (MBR)

```bash
mkswap /dev/[swap bölüm adı]
swapon /dev/[swap bölüm adı]
```

#### Root Bölümünü Bağla (MBR)

```bash
mount /dev/[root bölüm adı] /mnt
```

## Asıl Sistem Kurulumu

### Yansıları Reflector Aracını Kullanarak Güncelle

```bash
reflector -c Turkey -p http --save /etc/pacman.d/mirrorlist
```

- Kendiniz başka ülke eklemek isterseniz `-c` bayrağını tekrar kullanabilirsiniz.

### Ana Sistemi Kur

```bash
pacstrap /mnt base base-devel linux-lts linux-firmware linux-lts-headers nano intel-ucode
```

- `nano` yerine kendi editörünüzü seçebilirsiniz (örneğin `vim` ya da `vi`).
- `intel-ucode` paketini AMD işlemci kullanıyorsanız `amd-ucode` ile değiştirin.

### FSTab Oluştur

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

## Chroot'a Gir

```bash
arch-chroot /mnt
```

### Swap Dosyası Oluştur (UEFI için)

`count=4096` değerini RAM miktarınızın iki katı ile değiştirin. 16GB üstü RAM'iniz varsa gerek yok.

```bash
dd if=/dev/zero of=/swapfile bs=1M count=4096 status=progress
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

### Swap Dosyasını `/etc/fstab` Dosyasına Ekleme (UEFI için)

Alttaki satırı `/etc/fstab` dosyasının en altına ekleyin.

```
/swapfile none swap defaults 0 0
```

### Saati ve Tarihi Ayarla

```bash
ln -sf /usr/share/zoneinfo/Europe/Istanbul /etc/localtime
hwclock --systohc
```

Kendi saat diliminize göre `/Europe/Istanbul` kısmını değişebilirsiniz. Daha fazla bilgi için **[buraya](https://wiki.archlinux.org/title/installation_guide#Time_zone)** göz atın.

## Dili Ayarla

#### locale.gen Dosyasını Düzenle

```bash
nano /etc/locale.gen
```

Alttaki satırı bulup başındaki yorumu kaldırın.

```bash
#tr_TR.UTF-8 UTF-8
```

kaydet ve çıkın.

### Yerel Ayar Oluştur

```bash
locale-gen
```

### locale.conf Dosyasına Dili Ekle

```bash
echo LANG=tr_TR.UTF-8 > /etc/locale.conf
```

### vconsole'a Klavye Düzenini Ekle

```bash
echo "KEYMAP=trq" > /etc/vconsole.conf
```

## Bilgisayar Adını Ayarla

```bash
echo arch > /etc/hostname
```

`arch` kısmını bilgisayara koyacağınız isim ile değiştirin.

### hosts Dosyasını Düzenle

```bash
nano /etc/hosts
```

alttaki Satırları Ekle

```
127.0.0.1    localhost
::1          localhost
127.0.1.1    arch.localdomain arch
```

`arch` kısmını bilgisayara koyduğunuz isim ile değiştirin. Kaydedin ve çıkın.

### NetworkManager Kurup Aktifleştir

```bash
pacman -S networkmanager
systemctl enable NetworkManager
```

### Root Şifresini Ayarla

```bash
passwd
```

### GRUB Önyükleyicisini Kur

```bash
pacman -S grub
```

#### UEFI Sistem İçin

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
```

#### MBR Sistem İçin

```bash
grub-install /dev/[disk adı]
```

#### GRUB Yapılandırma Dosyasını Oluştur

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### Son Adımlar

```bash
exit
umount -a
reboot
```

## Şimdi Kurulu Olan Arch Linux Sisteminizi Açın.

### Root Olarak Giriş Yapın

### Yeni Kullanıcı Oluştur

```bash
useradd -mG wheel [kullanıcı adı]
```

`[kullanıcı adı]` kısmını kendi kullanıcı adınız ile değiştirin.

### Kullanıcı Şifresini Ayarla

```bash
passwd [kullanıcı adı]
```

### Wheel Grubunu Sudo Kullanılabilir Olarak Ayarla

```bash
EDITOR=nano visudo
```

Alttaki satırı bulup yorumu kaldırın.

```bash
#%wheel ALL=(ALL) ALL
```

kaydet ve çık.

### Root'tan Çıkış Yap

```bash
exit
```

## Kullanıcı Adın İle Giriş Yap

### Güncellemeleri Kontrol Et

```bash
sudo pacman -Syu
```

### XORG ve GPU Sürücülerini Kur

```bash
sudo pacman -S xorg [xf86-video-(gpu-markan)]
```

`(gpu-markan)` kısmına parantezler olmadan:

- NVIDIA GPUlar için `nvidia` ve `nvidia-settings` yazın. Daha eski GPUlar ya da daha fazla bilgi için [Arch Wiki - Nvidia](https://wiki.archlinux.org/index.php/NVIDIA) sayfasına göz atın.
- Yeni AMD GPUlar için `amdgpu` yazın.
- Eski Radeon GPUlar için (HD 7xxx ve altı) `ati` yazın.
- Gömülü Intel HD Graphics için `intel` yazın.

### Masaüstü Ortamı Kur

```bash
pacman -S [masaüstü ortamı]
```

| Masaüstü Ortamı | Paket İsmi  |
| --------------- | ----------- |
| GNOME           | gnome       |
| GNOME (minimal) | gnome-shell |
| KDE Plasma      | plasma      |
| LXDE            | lxde        |

Daha fazlası için **[bu sayfayı](https://wiki.archlinux.org/title/Desktop_environment)** ziyaret edin.

### Görüntü Yöneticisi Kur

```bash
sudo pacman -S [dm]
```

`[dm]` kısmına:

- Eğer GNOME kurduysanız `gdm` yazın. `sudo systemctl enable gdm` ile aktifleştirin.
- Eğer KDE Plasma kurduysanız `sddm` yazın. `sudo systemctl enable sddm` ile aktifleştirin.
- Eğer LXDE kurduysanız sadece `sudo systemctl enable lxdm` yazın. Eğer servis bulunamadı gibi bir hata alıyorsanız `lxdm` paketini kurup tekrar deneyin.

### Ses Sürücüleri

```bash
sudo pacman -S pulseaudio pulseaudio-alsa
```

### Yeniden Başlat

```bash
reboot
```

### Sonuç

Yeniden başlattıktan sonra her şey hazır bir şekilde arayüze ulaşmış olacaksınız. Deneyimi iyileştirmek amacıyla aşağıdaki bazı ekstra adımları da uygulayabilirsiniz.

## Ekstalar (isteğe bağlı)

### [Yay](https://github.com/Jguer/yay) Kur

YAY, **[AUR](https://aur.archlinux.org/)** için bir yardımcıdır.

```bash
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -sri
```

## PulseEffects

Eğer ekolayzır ayarlarına ihtiyacınız varsa PulseEffects aracını kullanabilirsiniz.

```bash
sudo pacman -S pulseeffects
# git sürümü için
yay -S pulseeffects-git
```

> Bu işlem ayrıca `pipewire-pulse` paketini kurup PipeWire yerine geçer.

#### Önerilen Yazı Tiplerini Kur

```bash
pacman -S ttf-dejavu ttf-droid noto-fonts noto-fonts-emoji ttf-roboto
```

## Performans İyileştirmeleri

### paccache

Pacman önbellek temizleyicisi.

```bash
sudo pacman -S pacman-contrib
```

El ile pacman önbelleğini temizlemek için:

```bash
sudo paccache -r
```

#### paccache İşlemini Otomatikleştirmek İçin

```bash
sudo mkdir /etc/pacman.d/hooks
sudo nano /etc/pacman.d/hooks/clean_cache.hook
```

Aşağıdaki kodu yaz

```
[Trigger]
Operation = Upgrade
Operation = Install
Operation = Remove
Type = Package
Target = *
[Action]
Description = Pacman önbelleği temizleniyor...
When = PostTransaction
Exec = /usr/bin/paccache -r
```

kaydet ve çık.
