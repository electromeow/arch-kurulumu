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
- Eğer Wi-Fi kullanıyorsanız `wifi-menu` yazarak Wi-Fi ağınıza bağlanabilirsiniz.
- Eğer `wifi-menu` sizde yok ise, `iwctl` ile Wi-Fi ağınıza bağlanabilirsiniz. (Altta kullanım var.)
- Eğer internete iki şekilde de bağlanamıyorsanız telefonunuzu bilgisayara bağlayıp USB bağlantı ayarlarından `Ağ Paylaşma` seçeneğini seçerek bağlanabilirsiniz.
- Eğer bağlandıysanız devam edelim.

### Iwctl ile Wi-Fi Ağına Bağlanmak
Öncelikle `iwctl` komutunu yazalım.

Şimdi bağlı olan adaptörleri listelemek için `device list` yazıyoruz. Böylece tüm liste size gelicektir. Adaptörün name kısmı büyük ihtimal `wlan0` olucaktır, ama farklı bir ismi varsa bu kısımdaki `wlan0` yerine sizinki ile değiştirebilirsiniz.

Sırada ağları bulmak var. `station wlan0 scan` ve sonra `station wlan0 get-networks` yazarak ağları listeleyelim.

Son olarak bu listeden kendi internetimizi bulup bağlanmak kaldı. Bunun için `station wlan0 connect <isim>` yazıp enter tuşuna basıyoruz. Bizden bir şifre isteyecek, Wi-Fi ağınızın şifresini yazarak devam edin ve artık büyük ihtimal bağlanmış olucaksınız. Artık `exit` yazarak iwctl içinden çıkabilir ve kuruluma devam edebilirsiniz.

### Sistem Tarihini Güncelle ve Zaman dilimini Türkiye olarak ayarla

```bash
timedatectl set-timezone Europe/Istanbul
timedatectl set-ntp true
```

<hr />

## Diski Kurulum İçin Ayarlama

> :warning: Disklerinizi yönetirken son derece dikkatli olun, verileriniz silinirse beni suçlamayın.\
> Eğer bir VirtualBox sanal makinesine kuruyorsanız, MBR için olan adımları izleyin disk bölümlemesi ve GRUB kurulumu için. Veya makine ayarları > sistem bölümünden EFI'yi etkinleştir seçeneğini işaretleyin ve UEFI için olan adımları izleyin.

## UEFI Sistem İçin

### Disk Bölümleme (UEFI için)

`gdisk` aracı ile diskimizde `EFI Boot, Swap ve Root` olmak üzere üç bölüm açacağız.

```bash
gdisk /dev/[disk ismi]
```

- [disk ismi] = bölümlenecek disk, `lsblk` ile diskinizi bulabilirsiniz.
- Swap bölümü RAM miktarınıza göre değişir. Swap bölümüne ne kadar ayırmanız gerektiğini şu şekilde belirleyebilirsiniz:
- 512MB ile 2GB arasında bir RAM varsa RAM miktarının 1.5 katı
- 2-16GB arasında bir RAM varsa RAM miktarı kadar
- 16GB üzerinde RAM bulunuyorsa 16GB
- swap alanı olmalıdır.

```
g = GPT partition tablosu oluşturun

n = Yeni bölüm
enter tuşuna bas = ilk bölüm
enter tuşuna bas = ilk sektör olarak
+512M = son sektör olarak (Boot bölüm boyutu)
ef00 = EFI bölüm tipi

n = Yeni bölüm
enter tuşuna bas = 2. bölüm
enter tuşuna bas = ilk sektör
+4G = son sektör olarak(Swap alanı boyutu. Siz bunu 4GB değil, RAM'inize bağlı olarak farklı bir değer de yapabilirsiniz)
8200 = Swap bölüm tipi

n = Tekrardan yeni bölüm
enter tuşuna bas = 3. bölüm
enter tuşuna bas = ilk sektör olarak
enter tuşuna bas = son sektör olarak [Root bölüm boyutu (kalan diski kullanarak)]
8300 ya da enter tuşuna bas = EXT4 Root bölüm tipi

w = kaydet ve çık
```

### Bölümleri Formatlama (UEFI için)

```
mkfs.fat -F32 /dev/[efi bölüm adı]
mkswap /dev/[swap bölüm adı]
mkfs.ext4 /dev/[root bölüm adı]
```

### Bölümleri Bağlama (UEFI için)

```
mount /dev/[root bölüm adı] /mnt
swapon /dev/[swap bölüm adı]
mkdir /mnt/boot/efi
mount /dev/[efi bölüm adı] /mnt/boot/efi
```

## MBR (BIOS) Sistem İçin

### Disk Bölümleme (MBR için)

`fdisk` aracı ile diskimizde `Boot, Swap ve Root` olmak üzere üç bölüm oluşturacağız.

```bash
fdisk /dev/[disk ismi]
```

- [disk ismi] = bölümlenecek disk, `lsblk` ile diskinizi bulabilirsiniz.
- Swap bölümü RAM miktarınıza göre değişir. Swap bölümüne ne kadar ayırmanız gerektiğini şu şekilde belirleyebilirsiniz:
- 512MB ile 2GB arasında bir RAM varsa RAM miktarının 1.5 katı
- 2-16GB arasında bir RAM varsa RAM miktarı kadar
- 16GB üzerinde RAM bulunuyorsa 16GB
- swap alanı olmalıdır.

```
o = DOS bölümleme tablosu oluşturun

n = Yeni bölüm
p = primary olarak seç
enter tuşuna bas = 1. bölüm
enter tuşuna bas = ilk sektör olarak
+600M = son sektör olarak(600MB boyut vermiş olduk. bu bölüm boot bölümümüz olacak)

n = Yeni bölüm
p = primary olarak seç
enter tuşuna bas = 2. bölüm
enter tuşuna bas = ilk sektör olarak
+4G = son sektör olarak(4GB yerine farklı miktarda bir yer de ayırabilirsiniz RAMinize göre. ben örnek olarak 4G yazdım.)

n = Yeni bölüm
p = primary olarak seç
enter tuşuna bas = 3. bölüm
enter tuşuna bas = ilk sektör olarak
enter tuşuna bas = son sektör olarak diskin kalanının tamamını kullan(bu bölüm root bölümü olacak)

t = disk tipini değiştir
2 = 2. bölüm olan swap için ayrılacak bölümü seç
82 = Swap bölüm olarak ayarla

w = değişiklikleri kaydet ve çık
```


### Bölümleri Formatlama, SWAP Oluşturup Root'u Bağlama (MBR)

#### Root ve Boot Bölümlerini EXT4 Olarak Formatlama (MBR)

```bash
mkfs.ext4 /dev/[root bölüm adı]
mkfs.ext4 /dev/[boot bölüm adı]
```

#### Swap Bölümünü Formatla (MBR)

```bash
mkswap /dev/[swap bölüm adı]
```

#### Bölümleri Bağla (MBR)

```bash
mount /dev/[root bölüm adı] /mnt
swapon /dev/[swap bölüm adı]
```

## Asıl Sistem Kurulumu

### Yansıları Reflector Aracını Kullanarak Güncelle

```bash
reflector -c Turkey -p http --save /etc/pacman.d/mirrorlist
```

- Kendiniz başka ülke eklemek isterseniz `-c` bayrağını tekrar kullanabilirsiniz.

### Ana Sistemi Kur

```bash
pacstrap /mnt base linux linux-firmware linux-headers nano intel-ucode sudo
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

Alttaki iki satırı bulup başlarındaki yorumları kaldırın. Sisteminizde İngilizce(zorunlu) ve kullanmak istediğiniz diğer dil paketleri(Türkçe gibi) bulunmalı.

```bash
#en_US.UTF-8 UTF-8
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
tr_TR.UTF-8 yerine İngilizce kullanmak isterseniz en_US.UTF-8 yazabilirsiniz.

### vconsole'a Klavye Düzenini Ekle

```bash
echo "KEYMAP=trq" > /etc/vconsole.conf
```
trq yerine siz isterseniz farklı bir klavye düzeni kullanabilirsiniz. Yani eğer İngilizce veya başka bir dilde klavye kullanıyorsanız trq yerine o dilin klavye düzenini yazmalısınız.

## Bilgisayar Adını Ayarla

```bash
echo "arch" > /etc/hostname
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
pacman -S efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
```

#### MBR Sistem İçin

```bash
grub-install /dev/[disk adı]
```
Burada disk adı derken, bölüm adı değil, direkt diskin adını veriyoruz.

#### GRUB Yapılandırma Dosyasını Oluştur

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### Son Adımlar

```bash
exit
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

Sudoers dosyasını açın.

```bash
EDITOR=nano visudo
```

Alttaki satırı bulup yorumu kaldırın.

```bash
# %wheel ALL=(ALL) ALL
```

kaydedin ve çıkın.

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

Sizin tercihiniz ama ben kendi düşüncelerimden bir tavsiye vereyim: minimal olsun diye gnome-shell indirmeyin. İçinde ayarlar menüsü veya terminal dahi gelmiyor bomboş bir ekrana bakıp kalıyorsunuz.\
Eğer KDE Plasma kullanmayı tercih edecekseniz, plasma paketinin yanında plasma-wayland-session ve kde-applications paketlerini de kurmalısınız. 
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

### Neofetch
Neofetch, terminal üzerinden sistemle ilgili pek çok bilgiyi öğrenmenizi sağlayan bir komut satırı aracıdır.\
Daha çok SS almak için kullanılır. İnternette gördüğünüz o terminalde dağıtımın logosu bulunan ve yanında sistemle ilgili parametreler bulunan [bunun gibi](https://covid.b-cdn.net/linux3/posts/manjaro-linux-vs-arch-linux/8768a55f1d51af01c1b37d234f4e3836d69408c482d7a50848fcb126e694a50f.png) ekran fotoğraflarındaki terminalde o görüntüyü elde etmek için genellikle kullanılır.
```bash
sudo pacman -S neofetch
```
komutunu kullanarak indirebilir ve terminale `neofetch` yazarak kullanabilirsiniz.


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

### Swappiness ayarı
Swappiness, kernel'ın RAM'den depolama birimine dosya taşıma alışkanlığının dengesiyle ilgili
bir sistem değişkenidir. Ne kadar yüksek olursa, sistem RAM'den depolama birimine o kadar sık veri taşır.\
RAM'de alan kalmadığında swap alanı RAM gibi kullanılır. Ama tabii bu çok sık olduğunda hızı düşüren bir durumdur
çünkü depolama birimleri olan HDD ve SSD'ler RAM'lerden kat kat yavaştır.\
Eğer çok düşük bir RAM'iniz yoksa swappiness değerini düşürmek sisteminizi çok daha hızlı hale getirecektir.\
Swappiness varsayılan olarak 60'tır. Örnek olarak bunu 10'a düşürebilirsiniz:
```
sudo sysctl vm.swappiness=10
```

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
