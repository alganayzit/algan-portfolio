---
title: "Spanning Tree Protocol (STP) und die Architektur der Layer-2-Schleifenvermeidung"
date: 2026-08-24
draft: false
tags: ["Networking", "STP", "RSTP", "MST", "Switching", "L2", "Security"]
categories: ["Networking"]
summary: "Von der Root-Bridge-Wahl über die Berechnung der Port-Kosten, von den STP-Timern bis zur schnellen Konvergenz von RSTP, von PVST+/MST bis zur Rolle von STP in modernen Spine-Leaf-Architekturen – die technischen Details hinter der Layer-2-Schleifenvermeidung."
---
![Stp](feature.png)
## Einführung

Um Redundanz in einem Layer-2-Netzwerk (Data Link) aufzubauen, müssen physisch mehr als ein Kabel oder eine Switch-Verbindung vorhanden sein. Anders als der IP-Header besitzt der Ethernet-Header jedoch kein TTL-Feld (Time to Live). Entsteht dadurch in einem redundanten Layer-2-Netzwerk eine Schleife (Loop), können Broadcast-Pakete endlos kreisen, die MAC-Adresstabellen beschädigt werden und das gesamte Netzwerk blockiert werden – ein sogenannter Broadcast Storm.

STP (IEEE 802.1D) und sein schnellerer Nachfolger RSTP (IEEE 802.1w) erkennen physische Schleifen im Netzwerk und versetzen bestimmte Ports logisch in einen blockierenden Zustand.

---

## 1. Der Wahlmechanismus der Root Bridge

Im Zentrum jeder STP-Topologie steht eine **Root Bridge**. Alle Switches im Netzwerk senden sich gegenseitig **BPDU-Pakete (Bridge Protocol Data Unit)**, und über diesen Austausch wird die Root Bridge gewählt.

Auswahlkriterium ist der Switch mit der niedrigsten **Bridge-ID (BID)**. Anders als oft vereinfacht dargestellt, besteht die BID nicht nur aus zwei, sondern aus drei Komponenten:

```
Bridge ID = Priority + Extended System ID (VLAN-ID) + MAC-Adresse
```

* **Priority:** Der Standardwert beträgt **32768** und kann in Schritten von 4096 manuell angepasst werden.
* **Extended System ID:** Da in PVST+-Umgebungen für jedes VLAN eine eigene STP-Instanz läuft, gibt dieses Feld an, zu welchem VLAN eine gesendete BPDU gehört, und wird in den Priority-Wert eingerechnet.
* **MAC-Adresse:** Sind die Priority-Werte gleich, gewinnt der Switch mit der niedrigsten MAC-Adresse.

In einem gut durchdachten Netzwerkdesign wird die Wahl der Root Bridge nie dem Zufall überlassen: Die Priority der Core-Switches (Backbone) wird manuell abgesenkt – etwa auf **4096** oder **8192** –, während ein zweiter Root-Bridge-Kandidat eine Stufe darüber erhält (z. B. **8192** oder **16384**).

---

## 2. Portrollen, Portzustände und Path Cost

Sobald die Root Bridge gewählt ist, erhält jeder Switch-Port im Netzwerk eine bestimmte Rolle:

* **Root Port (RP):** Auf jedem Switch außer der Root Bridge derjenige Port, der die Root Bridge über den kürzesten/günstigsten Pfad erreicht (genau einer pro Switch).
* **Designated Port (DP):** Der Port, der für die Weiterleitung des Traffics in einem bestimmten Netzwerksegment zuständig ist. Alle aktiven Ports auf der Root Bridge sind standardmäßig DPs.
* **Alternate-/Backup-Port (AP/BP):** Redundante Ports, die bewusst im Zustand Blocking/Discarding gehalten werden, um genau keine Schleife entstehen zu lassen.

"Günstigster Pfad" ist dabei keine vage Formulierung – STP berechnet dies numerisch über den **Path Cost**. Die aktualisierte (kurze) Kostentabelle nach IEEE 802.1D sieht so aus:

| Verbindungsgeschwindigkeit | Path Cost (STP, Kurzwert) |
| :--- | :--- |
| 10 Mbit/s | 100 |
| 100 Mbit/s | 19 |
| 1 Gbit/s | 4 |
| 10 Gbit/s | 2 |

Ein Switch berechnet für jeden möglichen Pfad zur Root Bridge die Gesamtkosten – die Summe der Cost-Werte aller Ports entlang dieses Pfads – und wählt den Port mit den geringsten Gesamtkosten als Root Port.

---

## 3. Portzustände: 802.1D STP vs. 802.1w RSTP

Beim klassischen STP kann es 30 bis 50 Sekunden dauern, bis ein Port in den aktiven Zustand (Forwarding) wechselt. RSTP verkürzt diese Zeit durch den **Proposal/Agreement**-Handshake-Mechanismus auf Millisekunden.

| 802.1D-STP-Zustand | 802.1w-RSTP-Zustand | MAC-Lernen | Traffic-Weiterleitung |
| :--- | :--- | :---: | :---: |
| Disabled | Discarding | Nein | Nein |
| Blocking | Discarding | Nein | Nein |
| Listening | Discarding | Nein | Nein |
| Learning | Learning | Ja | Nein |
| Forwarding | Forwarding | Ja | Ja |

RSTP führt außerdem zwei neue Konzepte bei den Portrollen ein: den **Edge Port** (entspricht PortFast – ein Port, der direkt mit einem Endgerät verbunden ist) und **bereits vorab berechnete Alternate-/Backup-Ports** – fällt ein Link aus, sucht RSTP anders als STP nicht erst nach einem Alternativpfad, sondern aktiviert sofort den bereits im Voraus bestimmten Backup-Port.

---

## 4. STP-Timer: Woher kommen die 30–50 Sekunden wirklich?

Die Langsamkeit des klassischen STP ist kein Zufall, sondern ergibt sich aus der Summe dreier Timer:

* **Hello-Timer (2 s):** Die Root Bridge sendet alle 2 Sekunden eine BPDU.
* **Max Age (20 s):** Empfängt ein Switch 20 Sekunden lang keine BPDU von seinem Nachbarn, geht er von einer Topologieänderung aus.
* **Forward Delay (2 × 15 s):** Ein Port verbringt jeweils 15 Sekunden in den Zuständen Listening und Learning.

In Summe: **Max Age (20 s) + 2 × Forward Delay (2 × 15 s) = 50 Sekunden** ist die Konvergenzzeit im ungünstigsten Fall. In besseren Szenarien, etwa beim Ausfall eines direkten Nachbarn, sinkt dieser Wert auf rund 30 Sekunden.

RSTP verzichtet vollständig auf diese Wartelogik: Benachbarte Switches führen einen **Proposal/Agreement**-Handshake durch, und sobald jeder Switch bestätigt hat, dass sein Nachbar synchron ist, wechselt der Port direkt in den Zustand Forwarding. Das Ergebnis ist in den meisten Topologien eine Konvergenzzeit von unter einer Sekunde.

---

## 5. Ein Konvergenzszenario: Was passiert, wenn ein Link ausfällt?

Betrachten wir eine Dreieckstopologie aus drei Switches (SW1 = Root Bridge, SW2 und SW3 jeweils miteinander und mit der Root Bridge verbunden). Der Link zwischen SW2 und SW3 befand sich im Zustand Alternate/Blocking.

1. **Erkennung des Linkausfalls:** Fällt die physische Verbindung zwischen SW1 und SW2 aus, erkennt SW2 dies sofort über das physische Link-Down-Signal.
2. **Topology Change Notification (TCN):** SW2 sendet in RSTP eine Topology-Change-BPDU direkt in Richtung Root Bridge (bzw. in klassischem STP die bekannte TCN-BPDU-Kette).
3. **Löschen der MAC-Tabelle:** Alle Switches, die die Topologieänderung empfangen, verwerfen die alten Pfade in ihrer MAC-Adresstabelle – bei STP nach Ablauf des Forward Delay, bei RSTP sofort.
4. **Neuberechnung:** SW2 erkennt, dass es die Root Bridge nun über SW3 erreichen kann; der Alternate Port zwischen SW2 und SW3 wird zum Root Port hochgestuft.
5. **Aktivierung des neuen Pfads:** Da dieser Port bei RSTP bereits im Voraus berechnet und "bereit" war, wechselt er innerhalb von Millisekunden in den Zustand Forwarding. Beim klassischen STP muss dieser Port hingegen erneut die Phasen Blocking → Listening → Learning → Forwarding durchlaufen – genau hier entsteht die bekannte Verzögerung von 30–50 Sekunden.

---

## 6. PVST+, Rapid PVST+ und MST

Die bisherige Beschreibung geht von einer einzigen STP-Instanz aus. In realen Cisco-Umgebungen kommen jedoch drei unterschiedliche Betriebsmodi zum Einsatz:

* **PVST+ (Per-VLAN Spanning Tree Plus):** Führt für jedes VLAN eine separate, unabhängige STP-Instanz aus. Dadurch kann pro VLAN eine unterschiedliche Root Bridge gewählt werden, was wiederum eine Lastverteilung des Traffics über verschiedene Links ermöglicht – VLAN 10 kann beispielsweise SW1 als Root Bridge nutzen, VLAN 20 hingegen SW2.
* **Rapid PVST+ (RPVST+):** Die PVST+-Variante mit der schnellen Konvergenzlogik von RSTP – der von Cisco heute standardmäßig empfohlene Modus.
* **MST (Multiple Spanning Tree – IEEE 802.1s):** In großen Umgebungen mit Hunderten von VLANs ist eine eigene Instanz pro VLAN CPU-technisch teuer. MST reduziert diese Last, indem mehrere VLANs zu einer einzigen STP-"Instanz" (einer Region) zusammengefasst werden.

---

## 7. Layer-2-Netzwerksicherheit und Port-Schutzmechanismen

STP verhindert nicht nur Schleifen – bei falscher Konfiguration kann es auch eine Sicherheitslücke öffnen. Ein böswilliges Gerät könnte BPDUs mit einer niedrigeren Priority senden und sich selbst als Root Bridge ausgeben. Um genau das zu verhindern, kommen folgende Schutzmechanismen zum Einsatz:

* **PortFast:** Überspringt an Ports, die mit Endgeräten wie Servern oder PCs verbunden sind, die Phasen Listening/Learning und lässt den Port sofort in den Zustand Forwarding wechseln.
* **BPDU Guard:** Trifft an einem mit PortFast konfigurierten Port dennoch eine BPDU ein (etwa weil ein Nutzer unerlaubt einen Switch anschließt), wird der Port sofort in den Zustand **err-disable** versetzt, um das Netzwerk zu schützen.
* **Root Guard:** Verhindert, dass ein nicht autorisierter Switch zur Root Bridge des Netzwerks wird. Trifft an einem geschützten Port eine "bessere" BPDU ein, wird dieser Port in den Zustand **root-inconsistent** versetzt und der Traffic unterbrochen.
* **Loop Guard:** Verhindert, dass ein Alternate-Port bei einem physischen Leitungsfehler (z. B. Verlust einer unidirektionalen Verbindung) fälschlicherweise in den Zustand Forwarding wechselt und dadurch eine Schleife erzeugt.

Zu dieser Familie gehören auch zwei historische, von Cisco vor RSTP entwickelte Funktionen, die speziell zur Verkürzung der 30–50-sekündigen Konvergenzzeit des klassischen STP dienten: **UplinkFast** (ermöglicht einem Switch beim Verlust seines direkten Nachbarlinks den Wechsel auf einen Backup-Uplink, ohne das Forward Delay abzuwarten) und **BackboneFast** (erkennt einen indirekten Linkausfall, ohne die Max-Age-Zeit abzuwarten). Mit der breiten Einführung von RSTP sind beide Funktionen weitgehend überflüssig geworden, da RSTP dieselbe schnelle Konvergenz standardmäßig bietet.

---

## 8. Zusammenhang mit EtherChannel

Zwei Switches, die physisch über mehr als ein Kabel verbunden sind, tragen aus Sicht von STP ein Schleifenrisiko in sich, weshalb einer dieser Links in den Zustand Blocking versetzt wird. **EtherChannel (Port Channel)** bündelt diese physischen Verbindungen zu einer einzigen logischen Schnittstelle. STP betrachtet einen Port-Channel als einen einzigen Port – es entsteht also keine Schleife, und alle gebündelten physischen Links bleiben gleichzeitig aktiv (lastverteilt). Dies ist der Standardweg, um Bandbreite zu erhöhen, ohne STP zu deaktivieren.

---

## 9. Ein einfaches CLI-Konfigurationsbeispiel

Auf Cisco IOS lassen sich die oben beschriebenen Mechanismen in etwa mit folgenden Befehlen umsetzen:

```
! Root-Bridge-Priority manuell absenken (auf einem Core-Switch)
Switch(config)# spanning-tree vlan 10 priority 4096

! Rapid-PVST+-Modus aktivieren
Switch(config)# spanning-tree mode rapid-pvst

! PortFast + BPDU Guard auf einem Endgeräte-Port
Switch(config)# interface fastEthernet 0/1
Switch(config-if)# spanning-tree portfast
Switch(config-if)# spanning-tree bpduguard enable

! Root Guard auf einem Uplink-Port aktivieren
Switch(config)# interface gigabitEthernet 0/1
Switch(config-if)# spanning-tree guard root

! Loop Guard global aktivieren
Switch(config)# spanning-tree loopguard default
```

---

## 10. Was ersetzt STP im modernen Data Center?

In klassischen dreistufigen (Access-Distribution-Core-) Campus-Netzwerken ist STP nach wie vor Standard. In modernen **Spine-Leaf**-Data-Center-Architekturen bringt STP jedoch naturgemäß ein Problem mit sich: Da es sich im Kern um ein Active-Passive-Design handelt, hält es einen Teil der redundanten Links dauerhaft im Zustand Blocking – ein erheblicher Teil der verfügbaren Bandbreite bleibt so ungenutzt.

Aus diesem Grund setzen große Data Center zunehmend auf Architekturen, die STP weitgehend deaktivieren oder eingrenzen:

* **vPC (Virtual Port Channel) / MC-LAG:** Lässt zwei physische Switches logisch wie einen einzigen Switch erscheinen und verhindert so, dass die Links zwischen ihnen von STP blockiert werden; alle Links bleiben Active-Active.
* **VXLAN / EVPN:** Layer-2-Segmente werden unabhängig von der physischen Topologie über ein Layer-3-IP-Fabric "getunnelt". Dadurch kann das Spine-Leaf-Fabric selbst als reines Layer-3-Netzwerk betrieben werden (alle Links über ECMP aktiv), wodurch STP innerhalb des Fabrics vollständig entfällt – STP bleibt nur noch ganz am Rand (Edge) als Sicherheitsnetz für die Segmente bestehen, in denen Endgeräte angeschlossen sind.

---

## Fazit

STP ist eine der grundlegendsten – und zugleich kritischsten – Sicherheits- und Stabilitätsschichten in Layer-2-Netzwerken. Von der Root-Bridge-Wahl über die Path-Cost-Berechnung bis hin zu den Timern und Port-Schutzmechanismen dient jede Komponente einem einzigen Zweck: sicherzustellen, dass kein Broadcast-Paket jemals endlos kreist. RSTP und MST bewahren diese Grundlogik, während sie Geschwindigkeit und Skalierbarkeit verbessern. Moderne Data-Center-Architekturen wiederum schaffen STP nicht vollständig ab, sondern beschränken es bewusst auf den Bereich, in dem es tatsächlich noch gebraucht wird – den Rand des Netzwerks –, während der Kern auf ein rein Layer-3-basiertes, Active-Active-Design umgestellt wird.
