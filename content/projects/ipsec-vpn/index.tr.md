---
title: "Site-to-Site IPsec VPN Tasarımı ve Uygulaması"
date: 2026-01-23
description: "Cisco IOS kullanarak güvenli olmayan bir Genel ISP altyapısı üzerinden HQ ve Branch ağları arasında güvenli bir IPsec VPN tünelinin tasarımı ve yapılandırılması."
featureImage: "featured.png"
showDate: true
showAuthor: false
showReadingTime: true
---

![IPSec](featured.png)

### Projeye Genel Bakış

Bu proje, Cisco IOS kullanarak güvenli olmayan bir Genel ISP altyapısı üzerinden iki kurumsal ağ (HQ ve Branch) arasında güvenli bir **Site-to-Site IPsec VPN** tünelinin tasarımını ve yapılandırmasını göstermektedir.

Bu projenin amacı, coğrafi olarak birbirinden ayrılmış iki LAN arasında güvenli bir iletişim kanalı sağlamaktır. Bir IPsec tüneli uygulayarak, iç trafiğin şifrelenmiş kalmasını ve İnternet Servis Sağlayıcısı (ISP) tarafından görünmez olmasını sağlıyoruz.

---

### 🛠️ Teknolojik Altyapı

* **IPsec (Internet Protocol Security):** Veri gizliliğini, bütünlüğünü ve kimlik doğrulamasını sağlar.
* **IKE Phase 1 (ISAKMP):** Güvenlik ilişkilerini müzakere eder ve güvenli bir yönetim düzlemi kurar.
* **IKE Phase 2 (IPsec SA):** Gerçek verilerin nasıl şifreleneceğini ve aktarılacağını tanımlar.
* **AES-256 & SHA:** Maksimum güvenlik için kullanılan gelişmiş şifreleme ve karma (hashing) standartları.
* **Cisco IOS Güvenlik Özellikleri:** `securityk9` teknoloji paketinin kullanılması.

---

### 📊 Adresleme Tablosu

| Cihaz | Arayüz | IP Adresi | Açıklama |
| :--- | :--- | :--- | :--- |
| **HQ-Router** | Gig0/0 | 1.1.1.1/30 | Dışa Bakan (İnternet) |
| **HQ-Router** | Gig0/1 | 192.168.10.1/24 | HQ LAN Ağ Geçidi |
| **ISP-Router** | Gig0/0 | 1.1.1.2/30 | HQ Servis Sonlandırma |
| **ISP-Router** | Gig0/1 | 2.2.2.2/30 | Branch Servis Sonlandırma |
| **BR-Router** | Gig0/0 | 2.2.2.1/30 | Dışa Bakan (İnternet) |
| **BR-Router** | Gig0/1 | 192.168.20.1/24 | Branch LAN Ağ Geçidi |

---

### 🔬 Doğrulama ve Çalışma Kanıtı

```text
Tünelin çalışır durumda olduğunu ve trafiğin şifrelendiğini kanıtlamak için 
aşağıdaki doğrulama adımları gerçekleştirilmiştir:

========================================================================
1. Phase 1 Durumu (ISAKMP SA)
========================================================================
'show crypto isakmp sa' komutu, yönetim tünelinin başarıyla kurulduğunu 
doğrular.

HQ# show crypto isakmp sa
IPv4 Crypto ISAKMP SA
dst             src             state          conn-id status
1.1.1.1         2.2.2.1         QM_IDLE           1001 ACTIVE

Beklenen Durum: QM_IDLE (Başarılı ve kimliği doğrulanmış oturum).


========================================================================
2. Phase 2 Veri Akışı (IPsec SA)
========================================================================
'show crypto ipsec sa' komutu, şifrelemenin gerçekleştiğini teknik olarak 
kanıtlar. Paket sayaçları izlenerek verilerin güvenli tünelden geçtiği 
doğrulanır.

HQ# show crypto ipsec sa
interface: GigabitEthernet0/0
    Crypto map tag: MYMAP, local addr 1.1.1.1

   protected vrf: (none)
   local  ident (addr/mask/prot/port): (192.168.10.0/255.255.255.0/0/0)
   remote ident (addr/mask/prot/port): (192.168.20.0/255.255.255.0/0/0)
   
   #pkts encaps: 45, #pkts encrypt: 45, #pkts digest: 45
   #pkts decaps: 45, #pkts decrypt: 45, #pkts verify: 45

İzlenecek Kritik Sayaçlar:
- #pkts encaps: Şifrelenen ve tünele gönderilen paket sayısı.
- #pkts decaps: Tünelden alınan ve şifresi çözülen paket sayısı.


========================================================================
3. ICMP Bağlantı Testi (Ping)
========================================================================
Güvenli hat üzerinden uçtan uca bağlantıyı test etmek amacıyla HQ-PC 
(192.168.10.x) cihazından Branch-PC (192.168.20.x) cihazına bir ping testi 
başlatılmıştır.

HQ-PC> ping 192.168.20.10

Pinging 192.168.20.10 with 32 bytes of data:
Reply from 192.168.20.10: bytes=32 time=12ms TTL=126
Reply from 192.168.20.10: bytes=32 time=10ms TTL=126

Ping statistics for 192.168.20.10:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)

Sonuç: Başarılı (İlk tünel anlaşmasından sonra %0 kayıp).
Doğrulama: Sürekli ping atılması, Phase 2'de belirtilen kapsülleme 
ve şifre çözme sayaçlarının artmasına neden olur.