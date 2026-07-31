---
title: "Merkezi Ağ İzleme ve Güvenlik Sıkılaştırma"
date: 2026-03-12
description: "Cisco IOS kullanarak Syslog, SNMP v2c protokolleri ve SSH güvenlik sıkılaştırması ile çok sahalı bir ağ izleme altyapısının tasarımı ve yapılandırılması."
featureImage: "feature.png"
showDate: true
showAuthor: false
showReadingTime: true
---

![Network Monitoring](featured.png)

### Projeye Genel Bakış

Bu proje, **Syslog** ve **SNMP** protokollerini kullanarak çok sahalı (multi-site) bir ağ altyapısı için merkezi bir izleme (monitoring) sisteminin kurulmasını ve Cisco IOS cihazları üzerinde siber güvenlik sıkılaştırma (hardening) en iyi pratiklerinin uygulanmasını göstermektedir.

Projenin temel amacı, kurumsal bir ağ altyapısındaki görünürlüğü (visibility) en üst düzeye çıkarmak, ağ genelindeki tüm kritik olayları tek bir noktadan analiz etmek ve yönetim hatlarını yetkisiz erişimlere karşı koruma altına almaktır.

---

### 🛠️ Teknolojik Altyapı

* **Syslog:** Port durumları, yapılandırma değişiklikleri ve güvenlik uyarıları gibi tüm kritik sistem olaylarını anlık olarak merkezi bir sunucuya iletir.
* **SNMP v2c (Simple Network Management Protocol):** Cihaz envanteri, donanım kaynakları ve sistem tanımlarının MIB Browser üzerinden uzaktan sorgulanabilmesini sağlar.
* **Hassas Olay Zamanlaması (Timestamps):** Olası bir siber olay anında adli analizlerin (forensic analysis) hatasız yapılabilmesi için günlük kayıtlarına milisaniye düzeyinde zaman damgaları ekler.
* **Güvenlik Sıkılaştırması (Hardening):** Yetkisiz erişim girişimlerini önlemek ve kaba kuvvet (brute-force) saldırılarını tespit etmek amacıyla uygulanan güvenlik katmanıdır.

---

### 📊 Adresleme ve Topoloji Tablosu

| Cihaz | Arayüz | IP Adresi | Açıklama |
| :--- | :--- | :--- | :--- |
| **Core_Router** | Gig0/0 | 192.168.10.1/24 | Merkez Ofis Ağ Geçidi |
| **Branch_Router** | Gig0/0 | 10.0.0.2/30 | Uzak Saha Ofis Arayüzü |
| **Management_Server** | Fast0/1 | 192.168.10.100/24 | Syslog & SNMP Servis Sunucusu |

---

### 🔬 Doğrulama ve Çalışma Kanıtı


Sistemlerin kararlı çalıştığını ve güvenlik politikalarının devrede olduğunu
kanıtlamak amacıyla uygulanan yapılandırma çıktıları ve test adımları aşağıdadır:

---
1. Merkezi Günlük Kaydı ve Hassas Zaman Damgası Yapılandırması

Ağdaki cihazların log verilerini merkezi sunucuya yönlendirmek ve zaman 
analizini doğrulamak için kullanılan Cisco IOS komutları:

```text
Core_Router# configure terminal
Core_Router(config)# logging 192.168.10.100
Core_Router(config)# service timestamps log datetime msec
Core_Router(config)# line console 0
Core_Router(config-line)# logging synchronous
```
Beklenen Durum: Bir arayüz kapandığında (shutdown), sunucunun Syslog 
ekranına milisaniye (msec) damgalı log mesajının anlık olarak düşmesi.


2. SNMP v2c Sorgu ve Erişim Yapılandırması

Cihaz bilgilerinin izleme sunucusuna sadece okunabilir (Read-Only) olarak 
sunulması ve kurumsal bilgilerin tanımlanması:
```text

Core_Router(config)# snmp-server community GuvenliToplulukSifresi RO
Core_Router(config)# snmp-server contact Algan Ayzit
Core_Router(config)# snmp-server location Merkez Veri Merkezi
```
Doğrulama: Management_Server üzerindeki 'MIB Browser' aracı çalıştırılıp 
'GuvenliToplulukSifresi' girildiğinde, sistem envanterinin başarıyla çekilmesi.

3. Cihaz Yönetim Hattı Güvenlik Sıkılaştırması (Hardening)

Güvensiz Telnet protokolünün kapatılması, SSH v2 aktivasyonu ve brute-force 
saldırılarını izlemek için giriş denemelerinin günlüklenmesi:
```text
Core_Router(config)# hostname Core_Router
Core_Router(config)# ip domain-name alganayzit.local
Core_Router(config)# crypto key generate rsa (Modulus: 1024)
Core_Router(config)# ip ssh version 2

Core_Router(config)# line vty 0 4
Core_Router(config-line)# transport input ssh
Core_Router(config-line)# login local
Core_Router(config-line)# exit

Core_Router(config)# login on-failure log
Core_Router(config)# login on-success log
Core_Router(config)# enable secret EncriptedSecretPassword123
```
Sonuç Kontrolü: Cihaza yapılacak hatalı bir SSH giriş denemesinde, Syslog 
ekranında anında bir kimlik doğrulama hatası (LOGIN-4-AUTH_FAIL) uyarısı üretilir.

{{< button href="https://github.com/alganayzit/Cisco-Network-Monitoring-Project" target="_blank" icon="github" >}}
Proje Kaynak Kodları (GitHub)
{{< /button >}}