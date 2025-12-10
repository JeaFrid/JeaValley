# JeaValley Proje İzinleri Üzerine Güvenlik Analizi


Bu doküman, JeaValley Android uygulamasının talep ettiği işletim sistemi izinlerinin neden istendiğini ve bu izinlerin hangi işlevleri desteklediğini açıklar. Amaç; sesli–görüntülü iletişim, kulaklık yönetimi, bildirimleşme ve medya paylaşımı özelliklerinin güvenli biçimde çalıştığını şeffaf şekilde göstermektir.

#### Aşağıda, ayrıca Güvenlik Mimarisi de yer almaktadır.

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




# Güvenlik Mimarisi ve İlkeleri

Bu doküman, uygulamamızın güvenlik yaklaşımını hem teknik bilgisi olmayan kullanıcıya, hem de teknik altyapıya meraklı geliştiricilere açıklamayı amaçlar.  
Amacımız; **hangi veriyi nasıl koruduğumuzu**, **hangi tehditlere karşı tasarlandığımızı** ve **hangi noktalarda sınırlarımız olduğunu** şeffaf bir şekilde anlatmaktır.

---

## 1. Genel Bakış (Herkes İçin Özet)

Uygulama tasarlanırken temel hedefimiz şuydu:

> “Kullanıcının verisi; ağ üzerinden geçerken, sunucuda saklanırken ve gerçek zamanlı iletilirken **izinsiz bir üçüncü taraf** tarafından okunamaz veya izinsiz değiştirilemez olsun.”

Bu amaçla:

- Uygulama ile sunucu arasındaki tüm kritik iletişim **uçtan uca şifrelenir**.
- Her istek için **tek seferlik, kısa ömürlü güvenlik jetonları** kullanılır.
- Gerçek zamanlı güncellemeler (canlı veriler) bile, klasik HTTP istekleriyle aynı şifreleme modeline uygun şekilde korunur.
- Sunucu üzerinde depolanan veriler, projelere göre **ayrılmış alanlarda**, ayrı erişim anahtarlarıyla yönetilir.
- Sunucuya gelen istekler, otomatik olarak **orantısız trafik / DDoS denemelerine karşı izlenir ve kısıtlanır**.

Bu sayede:

- Aynı ağdaki biri (örneğin açık Wi-Fi kullanırken) trafiği dinlese bile, **verilerin içeriğini göremez**.
- Tekrar oynatma (“replay”) ve basit “kopyala-yapıştır” saldırıları **zaman damgası ve tek kullanımlık jetonlar** sayesinde engellenir.
- Sunucu, sadece uygulamaya ait geçerli yetkilendirme anahtarlarıyla gelen istekleri kabul eder.

---

## 2. Güvenlik Tasarım İlkeleri

Sistem, aşağıdaki temel prensiplere göre inşa edilmiştir:

1. **Savunma derinliği (defense-in-depth):**  
   Tek bir katmana güvenmek yerine; şifreleme, token sistemi, oran sınırlama, proje bazlı ayrım ve kod gizleme gibi **birden fazla güvenlik katmanı** birlikte çalışır.

2. **Gizlilikten çok bütünlük ve yetki kontrolü:**  
   Trafik zaten şifreli; asıl odak, **veriye kimin erişebildiği ve neyi değiştirebildiği** üzerinedir.


3. **Tekrar oynatma ve zaman manipülasyonuna karşı dayanıklılık:**  
   Tüm kritik istekler; zaman damgalı, tekil kimlikli ve tek kullanımlık güvenlik işaretleri ile korunur.

4. **Hata durumunda dahi şifreli yanıt:**  
   Sunucu, hata üretse bile yanıtlarını **şifreli formatta** döndürür. Böylece saldırgan, hata mesajları üzerinden sistemin iç yapısını okuyamaz.

---

## 3. Mimarinin Yüksek Seviye Özeti

Sistem 3 ana bileşenden oluşur:

1. **İstemci Uygulama (Mobil + Web + Desktop)**
   - Kullanıcının gördüğü arayüzdür.
   - Veriyi sunucuya **şifrelenmiş paketler** halinde gönderir.
   - Sunucudan gelen yanıtları **istemci tarafında çözer** ve kullanıcıya gösterir.

2. **Güvenlik Katmanı / API Sunucusu**
   - Tüm isteklerin geçtiği merkezdir.
   - Yetkilendirme kontrolü, şifre çözme, veri doğrulama ve iş kuralları burada uygulanır.
   - İstek başına güvenlik kontrolleri (token, zaman, tekrar kontrolü) burada yapılır.

3. **Veri Katmanı**
   - Kalıcı veriler ilişkisel bir veritabanında saklanır.
   - Anlık, geçici veya kuyruklu veriler yüksek performanslı bir bellek veri deposunda tutulur.
   - Bu iki katman birlikte çalışarak; hem yüksek hız, hem de veri dayanıklılığı sağlar.

---

## 4. Şifreleme ve İletişim Güvenliği

### 4.1. İsteklerin Şifrelenmesi

- Uygulama ile sunucu arasındaki kritik trafiğin tamamı, **güçlü bir blok şifreleme algoritması** ile korunur.
- Bu algoritma:
  - Modern kriptografi standartlarıyla uyumludur.
  - Hem gizlilik (confidentiality), hem de bütünlük (integrity) sağlar.
- Her istek için:
  - Benzersiz bir **istek kimliği (request id)**,
  - Yüksek çözünürlüklü **zaman damgası**,
  - Rastgele üretilmiş bir **başlangıç değeri (nonce)** kullanılır.
- Bu sayede:
  - Aynı veriyi tekrar gönderseniz bile, ortaya çıkan şifreli paket **her seferinde farklı** olur.
  - Saldırgan, paketleri kaydetse bile daha sonra **aynı istekleri yeniden kullanamaz**.

### 4.2. Tek Kullanımlık Güvenlik Jetonları

- Önemli işlemler için, her istekten önce sunucudan **tek kullanımlık bir güvenlik jetonu** alınır.
- Bu jeton:
  - Kısa ömürlüdür.
  - Tek bir istek için geçerlidir.
  - Yanlış kullanıldığında veya süresi geçtiğinde reddedilir.
- Bu mekanizma:
  - **“Tekrar oynatma” (replay) saldırılarını** engeller.
  - Saldırgan paketleri ele geçirse bile, bunları yeniden kullanamamasını sağlar.

### 4.3. Yanıtların Şifrelenmesi

- Sunucunun verdiği yanıtlar da **istek ile aynı kriptografik anahtarlar üzerinden** şifrelenerek gönderilir.
- Yanıtın kimliği ve zaman bilgisi istemci tarafından kontrol edilir:
  - Yanıt, beklenen istekle eşleşmiyorsa kabul edilmez.
  - Yanıt içeriği değiştirilmişse, bütünlük kontrolü nedeniyle şifre çözme başarısız olur.

---

## 5. Kimlik Doğrulama ve Yetkilendirme

### 5.1. Proje Anahtarları ve Erişim Modeli

- Sunucu; her proje / uygulama için **farklı bir erişim anahtarı** kullanır.
- Bu anahtar:
  - Klasik “kullanıcı parolası” değildir; sunucunuzu “hangi uygulamanın” kullandığını temsil eder.
  - Sunucu tarafında kaydedilir ve doğrulanır.
- Tüm kalıcı veri işlemleri:
  - Bu proje anahtarına göre **izole edilir**.
  - Bir projenin verisi, başka bir projenin anahtarı ile yönetilemez.

### 5.2. Oturum ve Cihaz Yönetimi

- Uygulama tarafında, cihaz bazlı oturum yönetimi yapılır.
- Kullanıcı çıkış yaptığında:
  - Cihazdaki oturum bilgileri temizlenir.
  - Online / offline durumunuz veritabanına yansıtılır.
- Kullanıcı hesabının silinmesi durumunda:
  - İlgili tüm kullanıcı verileri uygulamaya özel veri deposundan kaldırılır (uygulamanın iş kurallarına göre).

### 5.3. Orantısız Trafiğe ve DDoS Denemelerine Karşı Koruma

Sunucuya gelen istekler:

- IP başına temel istatistiklerle izlenir.
- “Kısa sürede aşırı istek” veya “bilinçli keşif/deneme trafiği” olarak algılanan durumlarda:
  - İlgili adres belirli süreler için **otomatik olarak engellenir**.
- Bu mekanizma:
  - Hem sunucu kaynaklarını korur,
  - Hem de kaba kuvvet (brute force) denemelerinin etkisini azaltır.

---

## 6. Gerçek Zamanlı Bildirimler ve WebSocket Güvenliği

Sistem, gerçek zamanlı veri akışı için **kalıcı bağlantılar** (örneğin WebSocket) kullanır.

Bu katmanda:

- Bağlantı kurulurken, ilk aşamada **erişim anahtarı doğrulaması** yapılır.
- Sunucudan istemciye gönderilen her mesaj:
  - Aynı şifreleme modeli ile paketlenir,
  - Ortadaki bir dinleyici tarafından çözülemez.
- İstemciye ulaşan mesajlar:
  - İstemci uygulama içinde çözülür ve işlenir,
  - Şifre çözme başarısız olursa veya veri beklenen formatta değilse, mesaj atılır.

Sonuç olarak:

- Canlı odalar, anlık güncellemeler, bildirim akışları gibi tüm gerçek zamanlı özellikler, **API istekleri ile aynı güvenlik seviyesinde** çalışır.

---

## 7. Veri Saklama, Dosyalar ve Ayırma Mantığı

### 7.1. Uygulama Verileri

- Uygulama verileri; esnek bir **anahtar/etiket/değer** modeli ile saklanır.
- Her kayıt:
  - Bir projeye,
  - Bir alana (bucket),
  - Bir etikete (tag),
  - Ve bir veri gövdesine (JSON yapısı) sahiptir.
- Erişim kontrolü:
  - Proje anahtarı ile,
  - İlgili alan/etiket bağlamında yapılır.

### 7.2. Dosya Saklama

- Kullanıcıya ait dosyalar (örneğin görseller):
  - Proje anahtarına özel bir dizinde tutulur.
  - Uygulama tarafından indirilebilmesi için, sunucu tarafında ek erişim kontrolü yapılır.
- Koruma mantığı:
  - Her dosya isteği için, istemcinin yetkili bir proje anahtarı ile sunucuya başvurması gerekir.
  - İsteğin geçerli ve yetkili olmadığı durumlarda dosya içeriği gönderilmez.

---

## 8. İstemci İçindeki Sırların Korunması

İstemci uygulamalar (özellikle mobil ve masaüstü sürümler):

- Sunucu adresleri,
- Proje anahtarları,
- Bazı özel yapılandırma değerleri gibi parametrelere ihtiyaç duyar.

Bu değerleri karartırız ve depolarda şifreleyerek saklarız.


## 9. Kullanıcının Sorumluluğu ve Öneriler

Güvenlik, hem sunucu tarafının hem de kullanıcının ortak sorumluluğudur.

Kullanıcı olarak:

- Cihazınızı başkalarıyla paylaşmayın.
- Kullandığınız cihazın parolasını, kilit ekranını aktif tutun.
- Resmi uygulama mağazaları dışından bilinmeyen uygulamalar yüklemeyin.
- Mümkün olduğunca güncel işletim sistemi ve tarayıcı sürümlerini kullanın.
- Şüpheli e-posta, bağlantı veya mesajlara karşı dikkatli olun.

Sunucu tarafında:

- İş yükü arttıkça, ek güvenlik önlemleri planlanmakta ve mimari dönemsel olarak gözden geçirilmektedir.
- Yeni tehditler ortaya çıktıkça, sistemde iyileştirmeler yapılmaya devam edilecektir.

---

## 10. Sık Sorulan Sorular (SSS)

### “Verilerim ağ üzerinden şifresiz gidiyor mu?”

Hayır. Uygulama ile sunucu arasındaki kritik veriler, modern ve güçlü bir şifreleme algoritması ile korunur.  
Aynı ağdaki biri, trafiğinizi dinlese bile, içeriğini anlamlı şekilde okuyamaz.

---

### “Birisi Wi-Fi’mde oturup verilerimi çalabilir mi?”

Tek başına Wi-Fi trafiğini dinlemek, sistem tasarımı gereği verilerinizi okumaya yetmez.  
Paketlerin içeriği şifreli ve bütünlük kontrolüne tabi tutulur.

---

### “Bu sistem tamamen kırılamaz mı?”

Hiçbir ciddi yazılım projesi, “%100 kırılamaz” iddiasında bulunmaz.  
Bizim iddiamız:

- Güncel ve güçlü şifreleme yöntemleri kullanmak,
- Saldırganın işini zorlaştıracak çok katmanlı bir savunma sağlamak,
- Uygulama ölçeği büyüdükçe sistemi sürekli gözden geçirmek ve güçlendirmektir.

Mutlak güvenlik yerine, **makul ve modern tehditlere karşı güçlü bir savunma** sunmayı hedefliyoruz.

---

## 12. Geliştirme ve İyileştirme

Güvenlik, “bir kere yapılıp unutulan” bir özellik değildir; sürekli devam eden bir süreçtir.

- Mimarimiz, yeni özellikler eklendikçe gözden geçirilir.
- Gerek performans, gerek güvenlik açısından iyileştirmeler planlı olarak uygulanır.

Herhangi bir güvenlik şüpheniz, sorunuz veya öneriniz olursa, bizimle iletişime geçebilirsiniz.

> Güvenlik, birlikte inşa edilir.  
> Biz burada, kendi tarafımıza düşen sorumluluğu ciddiyetle alıyor ve kullanıcı verisinin korunmasını temel öncelik olarak görüyoruz.


