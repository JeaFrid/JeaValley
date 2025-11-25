# JeaValley Proje İzinleri Üzerine Güvenlik Analizi

Bu doküman, JeaValley Android uygulamasının talep ettiği işletim sistemi izinlerinin neden istendiğini ve bu izinlerin hangi işlevleri desteklediğini açıklar. Amaç; sesli–görüntülü iletişim, kulaklık yönetimi, bildirimleşme ve medya paylaşımı özelliklerinin güvenli biçimde çalıştığını şeffaf şekilde göstermektir.

---

## 1. Son Kullanıcı İçin Açıklama

JeaValley, oyuncuları bir araya getiren ve sesli/görüntülü görüşmeleri destekleyen bir iletişim uygulamasıdır. Mobil cihazlarda ihtiyaç duyduğumuz izinlerin sebepleri şöyledir:

### **Mikrofon (RECORD_AUDIO)**
Sesli konuşma yapabilmen için gereklidir. Mikrofon yalnızca bir aramaya katıldığında aktif olur. Hiçbir ses kaydı saklanmaz; iletişim gerçek zamanlıdır.

### **Kamera (CAMERA)**
Görüntülü görüşmeler veya profil fotoğrafı/video çekimleri için kullanılır. Kamera yalnızca sen izin verdiğinde açılır ve kapalıyken görüntü alınmaz.

### **Bluetooth / Bluetooth Connect / Bluetooth Scan**
Kablosuz kulaklık ve mikrofonlar ile görüşme yapabilmek için gereklidir. Yalnızca ses cihazlarını bulmak ve bağlantı kurmak amacıyla kullanılır; cihaz bilgileri kayıt altına alınmaz.

### **Bildirim Gösterme (POST_NOTIFICATIONS)**
Uygulama kapalıyken bile gelen arama veya mesajlardan haberdar olmanı sağlar. Bu izin istediğin zaman cihaz ayarlarından kapatılabilir.

### **Ses Ayarlarını Değiştirme (MODIFY_AUDIO_SETTINGS)**
Görüşme sırasında hoparlör/kulaklık geçişi, yankı azaltma ve ses optimizasyonu gibi işlemler için gereklidir. Değişiklikler sadece arama süresince geçerlidir.

---

## 2. Geliştiriciler İçin Açıklama

Aşağıdaki tablo, her iznin teknik gerekliliğini ve güvenlik önlemlerini özetler.  
Teknoloji isimleri genel kavramlarla ifade edilmiştir (ör. *Sesli ve Görüntülü Görüşme Altyapısı*, *Çağrı Yönetim Bileşeni*, *Bildirim Hizmeti*).

| İzin | Teknik Kullanım Senaryosu | İlgili Bileşenler (Soyut) | Güvenlik / Gizlilik Tedbirleri |
| --- | --- | --- | --- |
| `android.permission.RECORD_AUDIO` | Mikrofon verisinin gerçek zamanlı iletişim kanallarına aktarılması. Mute/Unmute işlemleri çağrı yönetim bileşeninde yapılır. | Sesli ve Görüntülü Görüşme Altyapısı, Çağrı Yönetim Bileşeni | Mikrofon sadece çağrı aktifken açılır. Veri uçtan uca şifrelenmiş aktarım kanallarında gönderilir; uygulama içi depolama yoktur. |
| `android.permission.CAMERA` | Görüntülü aramalar ve profil medyası için kamera akışı sağlanması. | Görüntü Kaynağı Yöneticisi, Çağrı Yönetim Bileşeni | Kamera erişimi tamamen kullanıcı etkileşimiyle tetiklenir. Kayıt tutulmaz. Kamera açık/kapalı durumu arayüzden izlenebilir. |
| `android.permission.BLUETOOTH` `android.permission.BLUETOOTH_CONNECT` `android.permission.BLUETOOTH_SCAN` | Kablosuz kulaklık/mikrofon gibi ses cihazlarının bulunması, bağlanması ve çağrı sırasında yönlendirilmesi. | Ses Aygıtı Yöneticisi, Platform Ses API Katmanı | Tarama yalnızca ses cihazı bağlantısı için yapılır. Cihaz listeleri saklanmaz ve üçüncü taraflarla paylaşılmaz. |
| `android.permission.POST_NOTIFICATIONS` | Arama ve mesaj bildirimlerinin Android 13+ üzerinde görüntülenmesi. | Bildirim Hizmeti, Arka Plan Olay Dinleyicisi | Bildirimler kullanıcı kontrolündedir. İzin kapatıldığında uygulama sessiz moda geçer. |
| `android.permission.MODIFY_AUDIO_SETTINGS` | Hoparlör/kulaklık yönlendirmesi, gürültü azaltma ve ses profili optimizasyonu. | Ses Yönetim Katmanı, Çağrı Denetim Arayüzü | Değişiklikler yalnızca görüşme süresince uygulanır ve işlem bittiğinde otomatik olarak eski değerlere dönülür. |

---

### 2.1 Sesli/Görüntülü Arama Akışı (Kavramsal)

1. Kullanıcı bir arama başlatır.  
2. Gerekli izinler kontrol edilir; yoksa sistem izin talebi gösterir.  
3. Mikrofon ve kamera kaynakları güvenli iletişim kanallarına bağlanır.  
4. Kullanıcının mute/kamera kapatma işlemleri yalnızca ilgili medya kanalını devre dışı bırakır.  
5. Arama sona erdiğinde tüm medya kaynakları serbest bırakılır ve cihaz ses ayarları eski değerlerine döner.

---

### 2.2 Bildirim Altyapısı (Kavramsal)

- Bildirimler yalnızca Android fiziksel cihazlarda çalışır.  
- Bildirim izni olmadan uyarılar gösterilmez.  
- Hiçbir bildirim verisi uygulama içinde saklanmaz; tüm tetikleyiciler gerçek zamanlı sunucu olaylarından gelir.

---

### 2.3 Bluetooth Yönetimi (Kavramsal)

- Android 12+ gereği Bluetooth işlemleri CONNECT ve SCAN izinleriyle ayrılmıştır.  
- Uygulama yalnızca ses cihazlarını bulmak için tarama yapar.  
- Konum bilgisi, geçmiş cihaz listesi ve benzeri hassas veriler toplanmaz.

---

## 3. Verilerin İşlenmesi ve Korunması

1. **Gerçek zamanlı medya aktarımı:** Ses ve görüntü verileri uçtan uca şifrelenmiş iletişim kanallarıyla iletilir ve sunucularda depolanmaz.  
2. **Yerel depolama:** Bildirim tercihleri ve ses cihazı seçimleri cihaz içinde güvenli biçimde saklanır. Mikrofon/kamera kayıtları cihazda tutulmaz.  
3. **Kullanıcı kontrolü:** Tüm izinler Android sistem ayarlarından istenildiği anda iptal edilebilir.  
4. **Platform güvenliği:** Web platformunda yerel API çağrıları hiç çalıştırılmaz; izinler yalnızca Android cihazlarda talep edilir.

---

## 4. Sık Sorulan Sorular (FAQ)

**“Mikrofon izni sürekli açık mı?”**  
Hayır. Mikrofon yalnızca bir aramaya katıldığında etkinleşir ve görüşme bittiğinde kapanır.

**“Bluetooth izinleri konumumu takip eder mi?”**  
Hayır. Tarama yalnızca ses cihazı bulmak için yapılır. Konum bilgisi toplanmaz.

**“Bildirim iznini kapatırsam ne olur?”**  
Arama ve mesaj bildirimleri görünmez; uyarıları yalnızca uygulamadayken alırsın.

**“Ses ayarlarının değiştirilmesi cihazıma zarar verir mi?”**  
Hayır. Ayar değişiklikleri geçicidir ve arama sona erince otomatik olarak geri alınır.

---

## 5. Sonuç

JeaValley, talep ettiği tüm Android izinlerini yalnızca sunulan özellikleri sağlamak için kullanır. Gereğinden fazla veri işlenmez, medya içerikleri saklanmaz ve tüm izinler kullanıcı kontrolündedir. Bu doküman, uygulamanın izin kullanımını şeffaf ve anlaşılır biçimde açıklamayı amaçlar. Sorularınız için destek kanallarımızla iletişime geçebilirsiniz.

## 6. Hesap Silme
JeaValley'de hesap silmek için destek kutusuna silmek istediğiniz hesap hakkında talepte bulunabilirsiniz.
