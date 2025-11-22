# JeaValley Gizlilik & Güvenlik Politikası

Bu doküman, JeaValley uygulamasının kullanıcı verilerini nasıl topladığını, hangi bilgilerin zorunlu veya isteğe bağlı olduğunu, verilerin nerede ve ne kadar süre saklandığını, kimlerin hangi verileri görebildiğini ve hangi güvenlik önlemlerinin alındığını açıklar. Doküman aynı zamanda güvenlik politikamızı ve sorumluluk sınırlamalarını kapsar.

---

## 1. Topladığımız Veriler

### 1.1 Zorunlu Veriler
JeaValley hesabı oluştururken veya aktif kullanırken aşağıdaki bilgiler zorunludur:

- **Hesap Kimliği:** Sistem tarafından otomatik üretilen benzersiz kullanıcı ID’si.
- **E-posta Adresi:** Hesap doğrulama, şifre sıfırlama ve resmi bildirimler için gereklidir.
- **Parola veya Yetkilendirme Bilgisi:** Şifrelenmiş biçimde saklanır. Bazı kurumsal hesaplar için OAuth tokenları kullanılabilir.
- **Cihaz Bilgileri (sınırlı):** Oturum güvenliği ve hata analizi için işletim sistemi sürümü, uygulama versiyonu, oturum zaman damgaları.
- **Ses/Görüntü Geçici Verileri:** Sesli veya görüntülü görüşmelerde kullanılan medya akışları yalnızca gerçek zamanlı aktarım sırasında işlenir; kalıcı olarak saklanmaz.

### 1.2 İsteğe Bağlı Profil Verileri
Aşağıdaki alanlar tamamen kullanıcının kontrolündedir ve “Profil” > “Edit Profile” ekranından güncellenebilir:

- Kullanıcı adı, görünen ad veya takma ad.
- Profil fotoğrafı, kapak görselleri.
- Biyografi, oyun geçmişi, takım bilgileri.
- Sosyal medya bağlantıları veya iletişim kanalları.
- Konum veya zaman dilimi bilgisi (isteğe bağlı).

Bu alanlar doldurulmadığında profilinizde boş görünür ve toplulukla paylaşılmaz.

---

## 2. Verilerin Saklanması ve Paylaşımı

- **Sunucu Konumu:** Veriler, ByBug altyapısında şifrelenmiş veri tabanlarında saklanır. Yedekler şifreli biçimde tutulur.
- **Saklama Süresi:** Hesap silme talebine kadar saklanır. Yasal zorunluluklar gerektiriyorsa bazı günlük verileri sınırlı süreyle tutabiliriz.
- **Günlük/Log Verileri:** Hata ayıklama için anonimleştirilmiş kayıtlar oluşturulur; IP adresleri veya cihaz tanımlayıcıları keskinleştirilmez.
- **Üçüncü Taraf Paylaşımı:** Verileriniz reklam amaçlı üçüncü taraflara satılmaz. Sadece hizmetin çalışması için zorunlu altyapı sağlayıcıları (ör. bildirim servisleri, medya iletim hizmetleri) ile sınırlı teknik veri paylaşılır. Bu servislerin tamamı için servislerin kendi politikaları geçerlidir.

### 2.1 Herkese Açık Bilgiler
Profilinizde paylaştığınız takma ad, profil fotoğrafı, biyografi gibi bilgiler topluluk içindeki diğer kullanıcılara görünür. Topluluk gönderileri ve yorumlar uygulama içindeki tüm kullanıcılara açıktır.

### 2.2 Kısıtlı Bilgiler
E-posta adresiniz, telefon numaranız, özel destek talepleri ve sistem tarafından tutulan oturum verileri yalnızca güvenlik ekibimiz tarafından erişilebilir ve topluluk üyeleriyle paylaşılmaz.

---

## 3. Güvenlik Politikası

JeaValley, kullanıcı güvenliğini aşağıdaki ilkelere dayandırır:

1. **En Az Yetki Prensibi:** Tüm servisler sadece gerekli olduğu kadar erişim yetkisine sahiptir.  
2. **Şifreleme:** Şifreler tek yönlü algoritmalarla saklanır. Medya akışları ve API çağrıları TLS üzerinden yapılır.  
3. **İzin Şeffaflığı:** Mobil istemcilerde kullanılan tüm işletim sistemi izinleri [security.md](https://github.com/JeaFrid/JeaValley/blob/main/security.md) dokümanında ayrıntılı biçimde açıklanır ve yalnızca gerekli olduklarında talep edilir.  
4. **Veri Bütünlüğü:** Kritik işlemler (şifre değişimi, cihaz ekleme) için ek doğrulama katmanları uygulanır.  
5. **Güncel Yazılım:** Sunucu ve istemci bileşenleri düzenli olarak güvenlik yamalarıyla güncellenir.  
6. **Denetim ve İzleme:** Olağan dışı oturum hareketleri tespit edildiğinde hesap güvenlik uyarıları gönderilir.

---

## 4. Kullanıcının Görev ve Sorumlulukları

- Hesap parolanızı üçüncü kişilerle paylaşmayın ve güçlü, benzersiz parolalar tercih edin.
- Bilinmeyen cihazlarda oturum açtıysanız çıkış yapmayı unutmayın.
- Şüpheli etkinlikleri veya güvenlik açıklarını resmi destek kanallarımıza bildirin.
- Profilinizi düzenlerken paylaştığınız bilgilerden doğan sonuçlar kullanıcı sorumluluğundadır; hassas verileri herkese açık alanlara eklememenizi öneririz.

---

## 5. Sorumluluk Reddi

- JeaValley, hizmet altyapısını güvenli tutmak için azami özeni gösterir; ancak kullanıcı hatalarından kaynaklanan (örneğin parolanın paylaşılması, kötü amaçlı yazılımlar) veri sızıntılarından sorumlu tutulamaz.
- Üçüncü taraf bağlantılar veya kullanıcı paylaşımları üzerinden ulaşılan içeriklerin doğruluğu JeaValley tarafından garanti edilmez. Bu içeriklerden doğan zararlar ilgili kullanıcıların sorumluluğundadır.
- Ücretsiz beta veya erken erişim sürümlerinde meydana gelebilecek veri kayıplarına ilişkin tazmin yükümlülüğü bulunmamaktadır.
- Hiçbir yazılımın yüzde yüz güvenli olmadığının bilincinde olup, JeaValley tarafında doğabilecek veri sızıntılarına karşı bir sorumluluğumuz bulunmamaktadır. Bu kapsamda tüm güvenlik önlemlerini hassasiyetle aldığımızı bildiririz. Ancak yine de bu güvenlik duvarları, diğer yazılımlarda olduğu gibi kırılamaz değildir. Hiçbir yazılım, yüzde yüz kırılamaz olamaz. Bilgilerinizi platforma eklerken, bunu göz önünde bulundurun.

---

## 6. İletişim

Gizlilik veya güvenlik politikasına ilişkin sorularınız için destek ekiplerimize [support@jeavalley.com](mailto:support@jeavalley.com) adresinden veya uygulama içindeki destek kutusu üzerinden ulaşabilirsiniz.

Bu politikayı zaman zaman güncelleyebiliriz. Önemli değişiklikleri uygulama içi duyurularla ve web sitemizde yayınlarız. Güncel politikayı düzenli olarak gözden geçirmeniz önerilir.

