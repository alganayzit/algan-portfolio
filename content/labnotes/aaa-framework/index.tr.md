---
title: "AAA Framework Nedir? Kurumsal Ağlarda Kimlik ve Erişim Yönetimi"
date: 2026-05-06
description: "Otel analojisiyle AAA (Authentication, Authorization, Accounting) mantığı ve TACACS+ vs RADIUS protokol savaşları."
showTableOfContents: false
---
![AAA Framework Şeması](feature.png)
Kurumsal ağ güvenliğinden bahsettiğimizde aklımıza hemen karmaşık güvenlik duvarları (firewall), şifreleme algoritmaları veya devasa erişim listeleri (ACL) gelir. Fakat tüm bu yapının tam merkezinde, ağ altyapısının omurgasını oluşturan üç temel kelime bulunur: **AAA (Authentication, Authorization, Accounting)**.

Peki nedir bu AAA? Teknik terimlerin arasında kaybolmadan önce, bunu bir mühendislik dökümanı gibi değil de, hepimizin bildiği şık ve yüksek güvenlikli bir **otele giriş yapma senaryosu** üzerinden ele alalım.

---

### 🔑 Otel Analojisi ve Arka Planda Neler Dönüyor?

#### 1. Authentication (Kimlik Doğrulama): "Sen kimsin?"
Otelin lobisine adım attınız ve resepsiyona yöneldiniz. Görevli size oda anahtarınızı vermeden önce ilk olarak ne ister? Tabii ki pasaportunuzu veya kimliğinizi. Buradaki amaç, rezervasyonu yapan kişinin gerçekten "siz" olduğunu doğrulamaktır.
* **Arka Planda Neler Dönüyor?:** Bir mühendisin bir router veya switch cihazına SSH ile bağlanmaya çalıştığı an karşısına çıkan kullanıcı adı ve şifre ekranıdır. Ağ, arka planda şu sorunun cevabını arar: *Bu kimlik bilgileri geçerli mi? Algan gerçekten Algan mı?* Küçük ağlarda bu bilgiler cihazın yerel veri tabanında (`local database`) saklanırken; kurumsal yapılarda AAA bu talebi merkezi bir sunucuya (Cisco ISE veya Aruba ClearPass gibi) paslar.

#### 2. Authorization (Yetkilendirme): "Burada ne yapmaya iznin var?"
Kimliğiniz doğrulandı ve resepsiyonist size dijital bir oda kartı teslim etti. Ancak bu kart oteldeki tüm kapıları açmaz. Kendi odanıza veya spor salonuna girebilirsiniz; fakat kral dairesine veya sistem odasına kartı okuttuğunuzda kapı açılmaz.
* **Arka Planda Neler Dönüyor?:** Cihaza başarıyla oturum açmış olmanız, orada her şeyi değiştirebileceğiniz anlamına gelmez. Cisco dünyasında 0 ile 15 arasında yetki seviyeleri (`privilege levels`) bulunur. AAA sayesinde **Rol Tabanlı Erişim Kontrolü (RBAC)** uygulayarak, ağa yeni başlayan bir stajyere sadece izleme komutları (`show`) çalıştırabileceği bir yetki basılırken, kıdemli bir mühendise tüm konfigürasyon (`config t`) yetkileri dinamik olarak tanımlanır.

#### 3. Accounting (Hesap Verebilirlik / Günlükleme): "Burada ne yaptın?"
Otelden ayrılma (check-out) vaktiniz geldi. Resepsiyon size detaylı bir fatura dökümü uzatır: Saat 21:00'de mini bardan ne alınmış, gece hangi film izlenmiş... Otel yönetimi güvenlik ve faturalandırma için attığınız her adımı kaydetmiştir.
* **Arka Planda Neler Dönüyor?:** Ağ mühendisliğinde bu adım tam bir dijital ayak izidir. Cihazda oturum açıldığında bir `Start`, kapatıldığında ise `Stop` paketi merkezi AAA sunucusuna gönderilir. Bir omurga switch durup dururken çöktüğünde log kayıtları incelenerek `reload` komutunu kimin, saat kaçta yazdığı saniyesi saniyesine tespit edilir. Amaç suçlu aramak değil; ağın denetlenebilirliğini (`auditing`) sağlamaktır.

---

### ⚔️ Protokol Savaşları: TACACS+ vs. RADIUS

Konsepti anladığımıza göre işin mutfağına geçebiliriz. Gerçek dünyada bu cihazları merkezi bir güvenlik sunucusuna bağlarken iki popüler ağ protokolü kullanırız. Hangisini seçeceğimiz, ağda neyi korumak istediğimize göre tamamen değişir.

| Özellik | TACACS+ (Cihaz Yöneticisi) | RADIUS (Ağ Kapısı Bekçisi) |
| :--- | :--- | :--- |
| **Birincil Kullanım** | Cihaz Yönetimi (Router/Switch Yönetimi) | Ağa Erişim Güvenliği (VPN kullanıcıları, 802.1X Wi-Fi) |
| **Şifreleme Güvenliği** | **Tüm paketi (veriyi) şifreler** | Sadece şifre (password) alanını şifreler |
| **Katman Tipi** | TCP (Port 49) kullanır - Güvenilir bağlantı | UDP (Port 1812/1813) kullanır - Daha hızlı, hafif |
| **Mimari Yapı** | AAA bileşenlerini (A, A, A) tamamen ayırır | Kimlik Doğrulama ve Yetkilendirmeyi birleştirir |
| **Komut Bazlı Kontrol** | Komut bazlı anlık yetkilendirme sunar (CLI için şarttır) | Detaylı CLI komut kontrolü yoktur |

#### Altyapı Yönetiminde Neden TACACS+ Tercih Ediyorum?
Eğer kurumsal bir ağdaki router ve switch mimarisini yönetiyorsam, benim için net kazanan her zaman **TACACS+** protokolüdür. Tüm paketi şifrelemesi ağ topolojimizin ve komut detaylarımızın havada (over the wire) dinlenmesini engeller. Daha da önemlisi, AAA bileşenlerini ayırdığı için, bir mühendis CLI ekranında her "Enter" tuşuna bastığında sunucuya gidip *"Bu adamın şu an bu komutu çalıştırma yetkisi var mı?"* diye anlık olarak sorabiliriz. RADIUS mimari yapısı gereği bu esnekliği ve keskin komut kontrolünü bu kadar verimli sağlayamaz.

---

### 🚀 Modern Dünyada AAA ve Zero Trust (Asla Güvenme, Her Zaman Doğrula)

Geleneksel ağ tasarımlarında AAA genellikle sadece şirket içi cihazlara bağlanan mühendisleri denetlemek için kullanılırdı. Ancak günümüzün **Zero Trust (Sıfır Güven)** mimarilerinde ağın sınırları ortadan kalktı. "Şirket ağının içindeyim, öyleyse güvenliyim" felsefesi artık tamamen tarih oldu.

Burada, portfolyomda yer alan bir diğer kritik proje olan **Site-to-Site IPsec VPN** mimarisini düşünebiliriz. Kurduğumuz o güvenli şifreli tüneller, verinin havada çalınmasını engeller. Peki o tünelin ucundan ağımıza sızmaya çalışan kişinin gerçekten bizim çalışanımız olduğundan nasıl emin oluyoruz?

İşte burada AAA devreye giriyor:
1. Şubedeki (Branch) veya uzaktan çalışan bir kullanıcı tünel üzerinden merkez ağa (HQ) erişmek istediğinde, VPN gateway talebi yakalar.
2. Kimlik bilgileri anında arkadaki merkezi AAA sunucusuna (RADIUS vasıtasıyla) paslanır.
3. Sunucu sadece şifreyi doğrulamakla kalmaz; kullanıcının bağlandığı cihazın şirket sertifikasına sahip olup olmadığını, güncel bir antivirüs barındırıp barındırmadığını kontrol eder (Context-Aware Authentication).
4. Eğer her şey yolundaysa, kullanıcıya sadece işini yapacağı kadar minimal bir ağ segmentine erişim izni (`Authorization`) verilir.

Sonuç olarak AAA; sadece router yönettiğimiz eski usul bir araç değil, modern siber güvenlik mimarilerinin, VPN tünellerinin ve kimlik yönetim sistemlerinin (IAM) tam kalbinde duran, vazgeçilmez bir güvenlik disiplinidir.