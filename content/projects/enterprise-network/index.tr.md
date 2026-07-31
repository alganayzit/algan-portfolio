---
title: "Kurumsal Ağ Tasarımı ve Simülasyonu"
date: 2026-01-18
description: "Yüksek kullanılabilirlik, dinamik yönlendirme ve güvenlik sınırlarını sergileyen kapsamlı bir kampüs ağı mimarisi."
showDate: true
showAuthor: false
showReadingTime: true
---

![Kurumsal Ağ Topolojisi](featured.png)

### Projeye Genel Bakış
Bu proje; ölçeklenebilirlik, yedeklilik ve yapısal güvenlik gibi kurumsal standartları karşılamak üzere tasarlanmış çok katmanlı bir kurumsal ağ mimarisinin simülasyonuna odaklanmaktadır. Çekirdek (core), dağıtım (distribution) ve erişim (access) katmanlarına sahip modern bir kampüs ağını modelleyerek, mantıksal sınırların mühendislik standartlarına uygun olmasını sağlar.

### Çekirdek Mimari ve Teknik Yaklaşım

#### 1. Yüksek Kullanılabilirlik ve Yedeklilik (HSRP)
Varsayılan ağ geçidi (default gateway) seviyesindeki tek nokta arızalarını (single point of failure) önlemek amacıyla, dağıtım katmanında **Hot Standby Router Protocol (HSRP)** uygulanmıştır.
* *Amaç:* Kritik VLAN'lar için otomatik ağ geçidi yedekliliği sağlayarak, bir katman-3 anahtar veya yönlendirici devre dışı kaldığında trafiğin kesintisiz akmasını hedefler.

#### 2. Dinamik Yönlendirme ve Ölçeklenebilirlik (OSPF)
Ağ genelindeki erişilebilirlik, Tek Alanlı **OSPF (Open Shortest Path First)** protokolü kullanılarak yapılandırılmıştır.
* *Amaç:* Bu yaklaşım, hızlı yakınsama (convergence) ve verimli yol seçimi sunarak ağın büyümesini kolaylaştırır. OSPF alanları ve hat maliyetleri, omurga hatlarındaki bant genişliği kullanımını optimize edecek şekilde optimize edilmiştir.

#### 3. Segmentasyon ve Güvenlik (VLAN'lar ve ACL'ler)
Yayın alanları (broadcast domains), kurumsal fonksiyonlara (örn. Yönetim, BT, Operasyon, Misafir) göre ayrılmış yerel **VLAN'lar** kullanılarak sınırlandırılmıştır.
* *Amaç:* VLAN'lar arası yönlendirme, en düşük yetki (least-privilege) erişim modelini uygulamak amacıyla Erişim Kontrol Listeleri (ACL'ler) ile sıkı bir şekilde denetlenir ve yetkisiz trafiğin hassas ağ bölümlerine ulaşması engellenir.
#### 4. Sınır Güvenliği ve İnternet Çıkışı (NAT)
Kamuya açık (public) IP alanından tasarruf etmek ve güvenli internet çıkışı sağlamak amacıyla, ağ sınırında **Network Address Translation (NAT)** — özellikle Port Address Translation (PAT) — yapılandırılmıştır.
* *Amaç:* İç IP şemasını dış dünyadan gizleyerek, özel adresleri genel adreslere dönüştürür ve böylece temel bir sınır savunma katmanı oluşturur.

#### 5. Kurumsal Mobilite ve Misafir Erişimi (Wi-Fi)
Kablosuz ağ bağlantısı kampüs tasarımına entegre edilerek, kurumsal iç trafik ile misafir kullanıcı trafiği birbirinden izole edilmiştir.
* *Amaç:* Kablosuz uç cihazların dinamik olarak kendi ilgili VLAN'larına aktarılması sağlanmış ve çekirdek kaynaklara yetkisiz erişimlerin önüne geçilmiştir.

---

### Kullanılan Teknolojiler
<div class="flex flex-wrap gap-2 my-4">
  <span class="bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-200 text-xs font-medium px-2.5 py-1 rounded border border-slate-200 dark:border-slate-700">Cisco IOS</span>
  <span class="bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-200 text-xs font-medium px-2.5 py-1 rounded border border-slate-200 dark:border-slate-700">OSPF v2</span>
  <span class="bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-200 text-xs font-medium px-2.5 py-1 rounded border border-slate-200 dark:border-slate-700">HSRP</span>
  <span class="bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-200 text-xs font-medium px-2.5 py-1 rounded border border-slate-200 dark:border-slate-700">VLAN Trunking (802.1Q)</span>
  <span class="bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-200 text-xs font-medium px-2.5 py-1 rounded border border-slate-200 dark:border-slate-700">Extended ACLs</span>
  <span class="bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-200 text-xs font-medium px-2.5 py-1 rounded border border-slate-200 dark:border-slate-700">Dinamik NAT / PAT</span>
  <span class="bg-slate-100 dark:bg-slate-800 text-slate-800 dark:text-slate-200 text-xs font-medium px-2.5 py-1 rounded border border-slate-200 dark:border-slate-700">Kablosuz Ağ (WLAN)</span>
</div>