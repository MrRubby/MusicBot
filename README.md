# IncCheat - Minecraft Anti-Hile Sistemi

IncCheat, Minecraft sunucuları için geliştirilmiş kapsamlı ve profesyonel bir anti-hile sistemidir. Modern hile türlerini tespit etmek ve engellemek için gelişmiş algoritmalar kullanır.

## Özellikler

### Combat Kontrolleri
- **KillAura**: Otomatik saldırı hilelerini tespit eder
- **AutoClicker**: Anormal tıklama hızlarını kontrol eder
- **Reach**: Menzil hilelerini tespit eder

### Hareket Kontrolleri
- **Speed**: Hız hilelerini kontrol eder
- **Fly**: Uçma hilelerini tespit eder
- **NoFall**: Düşme hasarı engelleyicileri tespit eder
- **Timer**: Oyun hızı manipülasyonlarını kontrol eder
- **Jesus**: Su üzerinde yürüme hilelerini tespit eder

### Dünya Kontrolleri
- **Scaffold**: Otomatik blok yerleştirme hilelerini kontrol eder
- **Nuker**: Hızlı blok kırma hilelerini tespit eder
- **XRay**: Şüpheli maden bulma davranışlarını izler
- **Freecam**: Duvardan geçme ve uzak etkileşim hilelerini tespit eder

### Kayıt Sistemi
- Şüpheli oyuncuların hareketlerini kaydeder
- Kayıtları geri oynatma özelliği
- Farklı oynatma hızları (0.5x, 1.0x, 2.0x)

## Komutlar

### Ana Komutlar
- `/ic reload` - Konfigürasyonu yeniler
- `/ic violations <oyuncu>` - Oyuncunun ihlallerini gösterir
- `/ic reset <oyuncu>` - Oyuncunun ihlallerini sıfırlar
- `/ic toggle <check>` - Belirli bir kontrolü açar/kapatır

### Kayıt Komutları
- `/replay list` - Mevcut kayıtları listeler
- `/replay play <oyuncu> [zaman]` - Kayıt oynatır
- `/replay stop` - Oynatmayı durdurur
- `/replay speed <0.5/1.0/2.0>` - Oynatma hızını ayarlar

## Konfigürasyon

Her kontrol için özelleştirilebilir ayarlar:
- Aktif/Pasif durumu
- Uyarı eşiği
- Otomatik kick eşiği
- Otomatik ban eşiği
- Kontrol-spesifik parametreler

## Yetkiler

- `inccheat.admin` - Tüm komutlara erişim
- `inccheat.notify` - Hile bildirimleri alma
- `inccheat.bypass` - Kontrolleri bypass etme
- `inccheat.playback` - Kayıt sistemine erişim

## Kurulum

1. Plugin dosyasını `plugins` klasörüne atın
2. Lisans anahtarınızı `config.yml` dosyasına girin
3. Sunucuyu yeniden başlatın
4. `plugins/IncCheat` klasöründeki konfigürasyon dosyalarını düzenleyin
5. `/ic reload` komutu ile değişiklikleri uygulayın

## Gereksinimler

- Minecraft 1.8 veya üzeri
- Spigot/Paper sunucu
- Java 8 veya üzeri

## Satın Alma

Eklentiyi satın almak için:
- Discord: [Link]
- MC-Market: [Link]
- SpigotMC: [Link]

## Destek

Premium destek için Discord sunucumuza katılın: [Link]

## Lisans

Tüm hakları saklıdır. Bu yazılım lisanslı bir üründür ve kaynak kodu paylaşılmamaktadır. 
