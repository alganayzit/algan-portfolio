---
title: "Mühendisler İçin Kapsamlı Cloud Computing 101 Rehberi"
date: 2026-08-01
draft: false
tags: ["Cloud", "AWS", "Networking", "DevOps", "Infrastructure", "Terraform", "Security"]
categories: ["Cloud Computing"]
summary: "Geleneksel veri merkezlerinden modern bulut mimarilerine derinlemesine geçiş. Mimari bileşenler, yazılım tanımlı ağ (SDN), VPC tasarımı, güvenlik ve Infrastructure as Code (IaC) prensipleri."
---
![Cloud](feature.png)
Günümüz yazılım ve altyapı dünyasında "Cloud" (Bulut Bilişim) artık sadece popüler bir teknoloji terimi değil; modern bilişim altyapılarının ve kurumsal sistemlerin omurgası haline gelmiş durumda. Ancak Cloud, popüler kültürde genellikle yüzeysel bir şekilde "başkalarının bilgisayarlarını kiralama hizmeti" olarak tanımlansa da, işin altyapı, ağ (networking) ve mühendislik boyutuna indiğimizde bundan çok daha karmaşık ve güçlü bir mimariyi ifade eder.

Bu rehberde, geleneksel On-Premise yapılardan modern Cloud mimarilerine geçiş mantığını, hizmet/dağıtım modellerini, bir altyapı mühendisi gözüyle Cloud ağ tasarımını (SDN/VPC), yüksek erişilebilirlik (HA) prensiplerini ve kaçırılmaması gereken güvenlik kurallarını derinlemesine ele alacağız.

---

## 1. On-Premise Dünyasından Cloud'a Dönüşüm

Geleneksel **On-Premise** (lokal veri merkezi) mimarilerinde bir sistem kurmak; fiziksel sunucu tedariki (procurement), kablolama, iklimlendirme, kesintisiz güç kaynakları (UPS), rack kabinet düzeni ve ağ donanımları (switch, router, firewall) gibi ciddi bir fiziksel operasyon ve süreç yönetimi gerektirir.

Bu geleneksel yaklaşımın en büyük kısıtları şunlardır:

* **CapEx (Capital Expenditure) Yükü:** Yüksek ilk kurulum maliyeti. Henüz bir projenin gerçek trafik yükü ve başarısı belli değilken devasa donanım yatırımları yapılması gerekir.
* **Kapasite Planlama Çıkmazı:** Anlık trafik artışlarında sistem yetersiz kalıp çökebilir (**under-provisioning**); veya en yüksek trafik anı düşünülerek yapılan donanım yatırımları, normal günlerde kaynakların %80'inin boşta kalmasına neden olur (**over-provisioning**).
* **Esneklik (Agility) Eksikliği:** Yeni bir sunucunun sipariş edilmesi, veri merkezine teslimi, montajı ve konfigürasyonu haftalar hatta aylar alabilir.

**Cloud Computing**, donanımı fiziksel bir varlık olmaktan çıkarıp bir **hizmet (utility)** ve **yazılım nesnesi** haline getirir. Tıpkı elektrik veya su şebekesi gibi: İhtiyacın kadar kaynağı anında oluşturur, kullandığın kadar öder (**Pay-as-you-go**), ve işin bittiğinde kaynağı tek bir API çağrısı ile yok edersin.

---

## 2. Cloud Hizmet Modelleri (XaaS)

Bulut servisleri, kullanıcıya sunulan soyutlama (abstraction) seviyesine ve yönetim sorumluluğuna göre üç ana kategoriye ayrılır:


| Hizmet Modeli | Sizin Sorumluluğunuz | Bulut Sağlayıcısının Sorumluluğu |
| :--- | :--- | :--- |
| **SaaS** | Sadece Veri ve Kullanımı | Uygulama, Runtime, OS, Donanım |
| **PaaS** | Uygulama ve Veri | Runtime, İşletim Sistemi, Sanallaştırma, Donanım |
| **IaaS** | İşletim Sistemi, Uygulama, Ağ Ayarları | Sanallaştırma, Donanım, Veri Merkezi |
### 🏢 IaaS (Infrastructure as a Service)
Altyapının en temel seviyesidir. Donanım ve sanallaştırma katmanı bulut sağlayıcısı (AWS, Azure, GCP) tarafından yönetilir; sanal sunucu (VM), disk depolama, işletim sistemi seçimi ve detaylı ağ yapılandırması tamamen sizin kontrolünüzdedir.
* **Kullanım Senaryosu:** Mevcut On-Premise sistemleri aynen buluta taşımak (Lift-and-Shift) veya tam kontrol gerektiren altyapılar.
* **Örnekler:** AWS EC2, Azure VMs, Google Compute Engine.

### 🛠️ PaaS (Platform as a Service)
İşletim sistemi güncellemeleri, güvenlik yamaları, çalışma zamanı (runtime: Node.js, Python, Java vb.) ve veritabanı motoru yönetimi bulut sağlayıcısına bırakılır. Geliştirici sadece kodunu yazar ve ortama gönderir.
* **Kullanım Senaryosu:** Sunucu yönetimiyle vakit kaybetmeden hızlıca ürün/uygulama canlıya almak isteyen yazılım ekipleri.
* **Örnekler:** AWS Elastic Beanstalk, Heroku, Google App Engine, Azure App Service.

### 📦 SaaS (Software as a Service)
Son kullanıcıya yönelik, web browser veya API üzerinden erişilen, tamamen yönetilen hazır yazılımlardır. Herhangi bir kodlama veya donanım yönetimi gerektirmez.
* **Örnekler:** Google Workspace, Microsoft 365, Salesforce, Jira.

---

## 3. Bulut Dağıtım Modelleri (Deployment Models)

* **Public Cloud:** Kaynakların genel internet üzerinden üçüncü taraf sağlayıcılar (AWS, Azure, GCP) tarafından herkese sunulduğu model. Yüksek ölçeklenebilirlik sağlar.
* **Private Cloud:** Sadece tek bir kuruma özel, şirket içi veri merkezinde (On-Premise) veya izole edilmiş özel sunucularda çalışan bulut yapısı (OpenStack, VMware Cloud Foundation).
* **Hybrid Cloud:** On-Premise altyapı ile Public Cloud'un güvenli bir şekilde birbirine bağlanarak hibrit çalıştırıldığı model. Finans, sağlık ve bankacılık gibi hassas veri barındıran ancak Public Cloud'un esnekliğinden de yararlanmak isteyen kurumların tercihidir.
* **Multi-Cloud:** İki veya daha fazla public cloud sağlayıcısının (örneğin AWS + Google Cloud) aynı anda kullanılarak bağımlılığın (**vendor lock-in**) azaltıldığı ve en iyi servislerin kombine edildiği mimari.

---

## 4. Mühendislik Perspektifi: Cloud ve Software-Defined Networking (SDN)

Birçok kişinin düştüğü yanılgı, Cloud'un sadece sanal sunuculardan ibaret olduğudur. Gerçekte **sağlam ve izole edilmiş bir ağ tasarımı olmadan hiçbir bulut mimarisi güvenli çalışamaz.** Fiziksel dünyadaki VLAN, subnet, routing table, NAT ve firewall kavramları Cloud ortamında **Software-Defined Networking (SDN)** ile kodlanabilir yapılara dönüşür.

### Virtual Private Cloud (VPC) Tasarımı
VPC, Public Cloud içinde kurumunuza özel tahsis edilmiş, mantıksal olarak izole edilmiş sanal ağ ortamıdır.

1. **Subnet Hiyerarşisi:**
   * **Public Subnet:** Doğrudan Internet Gateway'e (IGW) bağlı olan ve dış dünyadan erişim alması gereken bileşenlerin (Load Balancer, Bastion Host vb.) konulduğu ağ bloğu.
   * **Private Subnet:** Dış internetten doğrudan erişilemeyen, sadece iç ağdan ulaşılan ve internete çıkışını bir NAT Gateway üzerinden yapan blok (Uygulama sunucuları, Microservice'ler).
   * **Database Subnet:** İnternet erişimi tamamen kapalı, sadece Private Subnet'ten gelen isteklere cevap veren tam izole blok.

2. **Güvenlik Katmanları (Defense in Depth):**
   * **Security Group (SG):** Sanal sunucu (ENI) seviyesinde çalışan, **Stateful** (durum bilgisi tutan) güvenlik duvarı. Gelen trafiğe izin verildiğinde dönüş trafiğine otomatik izin verilir.
   * **Network ACL (NACL):** Subnet seviyesinde çalışan, **Stateless** (durum bilgisi tutmayan) güvenlik katmanı. Hem Gelen (Inbound) hem Giden (Outbound) kuralların açıkça tanımlanması gerekir.

```text
[ Internet ]
     ↓
[ Internet Gateway ]
     ↓
[ Public Subnet ]    ──> (Load Balancer / Security Group)
     ↓
[ NAT Gateway ]
     ↓
[ Private Subnet ]   ──> (App Servers / Security Group)
     ↓
[ Database Subnet ]  ──> (Isolated Database Cluster)
```

  ### On-Premise ile Cloud Bağlantı Yöntemleri
Hibrit mimarilerde lokal veri merkeziniz ile bulut VPC'nizi konuşturmak için iki temel yöntem kullanılır:
* **Site-to-Site IPsec VPN:** İnternet üzerinden şifreli tünel kurularak yapılan, kurulumu hızlı ve maliyet odaklı bağlantı.
* **Dedicated Direct Connection (AWS Direct Connect / Azure ExpressRoute):** Müşteri veri merkezi ile bulut sağlayıcısının PoP noktası arasında interneti tamamen bypass eden, sabit hat hızlı (1Gbps - 100Gbps), düşük gecikmeli (low-latency) fiziksel fiber hat.

---

## 5. Yüksek Erişilebilirlik (High Availability) ve Esneklik (Elasticity)

Cloud mimarisini geleneksel yapılardan ayıran en temel iki mühendislik prensibi şunlardır:

### 🔄 Auto Scaling & Elasticity
Sistem trafiği arttığında sunucuların işlemci (CPU) veya bellek (RAM) kapasitesini artırmak **Vertical Scaling (Scale Up)** iken; sisteme aynı özelliklerde yeni sunucu örnekleri eklemek **Horizontal Scaling (Scale Out)** olarak adlandırılır. 

Cloud ortamında **Auto Scaling Group (ASG)** yapısı sayesinde, örneğin CPU kullanımı %70'i aştığında mimari otomatik olarak yeni EC2/VM örnekleri ayağa kaldırır ve trafik düştüğünde bunları kapatır.

### 🌐 Multi-AZ ve Multi-Region Mimarisi
* **Availability Zone (AZ):** Bir bölgedeki (Region) birbirinden bağımsız güç, iklimlendirme ve ağ altyapısına sahip fiziksel veri merkezleridir.
* **Yüksek Erişilebilirlik (HA):** Bir sunucunun çökmesi durumunda sistemin kesintisiz çalışması için uygulamalar en az 2 farklı AZ'ye dağıtılmış subnet'lerde çalıştırılır ve önlerine bir **Application Load Balancer (ALB)** koyulur.

---

## 6. Güvenlik ve Paylaşılan Sorumluluk Modeli (Shared Responsibility Model)

Cloud güvenliğinde yapılan en büyük stratejik hata "Her şeyi bulut sağlayıcısına taşıdık, güvenlik artık onların işi" yaklaşımıdır. AWS ve Azure gibi devler **Shared Responsibility Model** ilkesiyle çalışır:

* **Security OF the Cloud (Bulut Sağlayıcısının Sorumluluğu):** Fiziksel veri merkezlerinin güvenliği, donanım arızaları, hypervisor katmanı, fiziksel ağ kablolaması ve altyapı tesis güvenliği.
* **Security IN the Cloud (Müşterinin Sorumluluğu):** Sanal sunucudaki işletim sistemi güncellemeleri, IAM (Identity and Access Management) kullanıcı yetkileri, firewall kuralları, verilerin şifrelenmesi (Encryption at rest & in transit) ve uygulama kodu güvenliği.

---

## 7. Kod Olarak Altyapı (Infrastructure as Code - IaC) ve FinOps

Modern Cloud yönetiminde sunucuları veya ağları konsol üzerinden elle tıklayarak oluşturmak ("ClickOps") sürdürülebilir değildir ve insan hatasına açıktır.

### Infrastructure as Code (IaC)
Altyapının tamamının (VPC, Subnet, EC2, Load Balancer vb.) kod dosyaları ile tanımlanmasıdır. 
* **Terraform / OpenTofu:** Bildirimsel (declarative) dili ile tüm bulut sağlayıcılarında altyapıyı kodla yönetmeyi sağlar.
* **Avantajları:** Versiyon kontrolü (Git ile takip), tekrarlanabilirlik (aynı altyapıyı saniyeler içinde test ortamına kurabilme) ve felaket kurtarma (Disaster Recovery) kolaylığı.

### FinOps ve Maliyet Yönetimi
Cloud esneklik sağlarken, kontrolden çıkan bütçelere de sebep olabilir. **FinOps**, mühendislik ve finans ekiplerinin birlikte çalışarak maliyetleri optimize etmesidir:
* Kullanılmayan boşta duran (idle) kaynakların tespiti ve silinmesi.
* 7/24 çalışan kararlı yükler için **Reserved Instances (RI)** veya **Savings Plans** kullanarak %60-70 varan indirimler elde edilmesi.
* Geçici işler için **Spot Instance** kullanımı.

---

## 8. Sonuç ve Gelecek Vizyonu

Cloud Computing, altyapı mühendisliğini fiziksel donanım bağımlılığından tamamen kurtarıp **Yazılım Tanımlı Altyapı (SDN & IaC)** dönemine taşımıştır. Günümüzde ağ mühendisliği, sistem yönetimi ve yazılım geliştirme rolleri **DevOps, SRE (Site Reliability Engineering) ve Cloud Engineering** disiplinleri altında birleşmektedir.

Bu dünyada yer almak isteyen bir mühendis için; ağ prensiplerini (TCP/IP, Routing, DNS), sanallaştırma/konteyner teknolojilerini (Docker, Kubernetes) ve Infrastructure as Code araçlarını (Terraform) Cloud prensipleriyle harmanlamak en kritik adımdır.

Öğrenim sürecinde teoride kalmayıp pazar payı yüksek olan **AWS** veya **Azure** üzerinde ücretsiz katman (Free Tier) hesapları açarak hands-on projeler üretmek, bu ekosisteme hakim olmanın en kestirme yoludur.