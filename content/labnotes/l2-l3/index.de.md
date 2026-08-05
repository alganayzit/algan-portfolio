---
title: "Layer 2 vs. Layer 3 Switching: Die technische Logik hinter Netzwerksegmentierung und Paketfluss"
date: 2026-08-05
draft: false
tags: ["Networking", "L2", "L3", "VLAN", "Routing", "Switching", "Security"]
categories: ["Networking"]
summary: "Die hardwareseitigen und logischen Grenzen eines flachen Layer-2-Netzwerks, die Funktionsweise von Layer-3-Switching (L3 Switching) und die Transformation, die ein Paket beim Übergang zwischen den beiden Schichten durchläuft – aus technischer Perspektive."
---

![L2-L3](feature.png)

Eine der grundlegendsten architektonischen Entscheidungen beim Netzwerkdesign ist, wie viel Traffic auf Layer 2 (Data Link) gehalten und wann er auf Layer 3 (Network) angehoben wird. Für Einsteiger oder alle, die sich mit Infrastrukturarchitektur beschäftigen, stellt sich dabei immer wieder dieselbe Frage: Wenn sich die Geräte im selben Gebäude oder sogar am selben Switch befinden, warum lässt man dann nicht einfach alles in einem einzigen, flachen Layer-2-Netzwerk? Warum VLANs anlegen und Layer-3-Switching- sowie Routing-Schichten dazwischenschalten?

In diesem Artikel betrachten wir aus technischer Perspektive die hardwareseitigen und logischen Grenzen eines flachen Layer-2-Netzwerks, die Funktionsweise von Layer-3-Switching (L3 Switching) sowie die Transformation, die ein Paket durchläuft, wenn es zwischen den beiden Schichten wechselt.

---

## 1. Die Grenzen eines einzigen Layer-2-Netzwerks: Broadcast und Hardwarelast

Befinden sich alle Geräte eines Netzwerks in derselben Layer-2-Domäne (Broadcast Domain), erreicht jedes Broadcast-Paket, das ein Gerät versendet – etwa ARP-Anfragen oder DHCP-Discovery-Pakete –, sämtliche anderen Geräte im Netzwerk.

In einem unkontrolliert wachsenden Layer-2-Netzwerk entstehen dadurch zwei grundlegende Probleme:

* **Broadcast-Stürme und CPU-Last:** Mit steigender Geräteanzahl im Netzwerk wächst die Zahl der Broadcast-Pakete überproportional. Da die Netzwerkkarten diese Pakete verarbeiten müssen, steigt sowohl bei Endgeräten als auch bei Switches die Prozessorlast.
* **Überfüllung der MAC-Adresstabelle (CAM Table):** L2-Switches leiten Pakete anhand ihrer MAC-Adresstabellen weiter. Mit wachsendem Netzwerk stoßen diese Tabellen an ihre Kapazitätsgrenzen, und die hardwareseitigen Suchzeiten verlängern sich.

Der Weg, diese Grenzen zu überwinden, besteht darin, das Netzwerk in logische Segmente – also VLANs – zu unterteilen. Sobald VLANs eingerichtet sind, stellt sich jedoch die Frage, wie diese unterschiedlichen Segmente miteinander kommunizieren sollen.

---

## 2. Die Logik von Layer-2- vs. Layer-3-Switching

Beim Layer-2-Switching läuft der Prozess vollständig über MAC-Adressen und Ethernet-Frames ab. Befinden sich die Geräte im selben Subnetz, lernt das Quellgerät die MAC-Adresse des Zielgeräts per ARP, und der Switch kann diese Weiterleitung hardwareseitig – über ASIC-Chips – mit Leitungsgeschwindigkeit durchführen.

Beim Layer-3-Switching hingegen läuft der Prozess über IP-Adressen und Pakete ab. Wenn Geräte in unterschiedlichen VLANs oder Subnetzen miteinander kommunizieren wollen, reicht Layer-2-Switching allein nicht mehr aus. Hier kommen die Switched Virtual Interface (SVI) und die IP-Routing-Mechanismen ins Spiel.

---

## 3. Die Grenze zwischen klassischem Router und L3-Switch

Wenn bereits ein Router vorhanden ist, warum setzt man dann einen Layer-3-Switch (L3-Switch) ein? Oder umgekehrt: Wenn ein L3-Switch vorhanden ist, wozu braucht man dann noch einen Router?

Auch wenn beide Geräte IP-Routing durchführen können, dienen ihre Architektur und Hardwareauslegung unterschiedlichen Zwecken:

* **L3-Switch (Inter-VLAN-Routing):** Trifft die Routing-Entscheidung nicht softwareseitig über die CPU, sondern direkt hardwareseitig über ASIC- oder CEF-Tabellen (Cisco Express Forwarding). Dadurch kann er den Traffic zwischen VLANs im lokalen Netzwerk mit Leitungsgeschwindigkeit (Wire-Speed) im Gigabit-/Terabit-Bereich und extrem niedriger Latenz weiterleiten.
* **Router (Edge / WAN-Grenze):** In der Regel dafür ausgelegt, komplexere Routing-Protokolle (z. B. BGP), Wide-Area-Network-Technologien (WAN), NAT (Network Address Translation), tiefgreifende Sicherheitsfunktionen sowie VPN-Tunneling-Prozesse abzuwickeln. Die Portdichte ist geringer als bei einem L3-Switch, dafür sind die Paketverarbeitungsfähigkeiten flexibler.

Kurz gesagt: Das schnelle Routing zwischen VLANs innerhalb eines Gebäudes oder Rechenzentrums übernimmt der L3-Switch, während der Router die Grenze zur Außenwelt bzw. zu anderen Standorten verwaltet.

---

## 4. Netzwerksegmentierung: Das Gleichgewicht zwischen Sicherheit und Performance

Der größte Vorteil von Layer-3-Switching ist die Netzwerksegmentierung. Wird das Netzwerk in kleinere Broadcast-Domänen unterteilt, gewinnt man nicht nur an Performance – gleichzeitig werden auch klare Sicherheitsgrenzen gezogen.

Durch die Definition von Access Control Lists (ACLs) auf den L3-Switches oder Firewall-Schichten zwischen unterschiedlichen VLANs lassen sich Regeln wie "Das HR-VLAN darf nur auf bestimmte Dienste im Server-VLAN zugreifen, während IoT-Geräte überhaupt kein internes Netzwerk erreichen dürfen" logisch durchsetzen.

---

## 5. Paketflussanalyse: Was passiert, wenn ein Paket die Schicht wechselt?

Um die Architektur vollständig zu verstehen, lohnt es sich, den Weg eines Pakets Schritt für Schritt nachzuvollziehen. Ein Beispiel: Computer A in VLAN 10 möchte ein Datenpaket an Server B in VLAN 20 senden. Dabei passiert Folgendes:

1. **Subnetzprüfung:** Computer A vergleicht seine eigene IP-Adresse und Subnetzmaske mit der Ziel-IP-Adresse von Server B und erkennt, dass sich dieser in einem anderen Netzwerk befindet.
2. **Weiterleitung an das Default Gateway:** Das Paket wird nicht direkt an die MAC-Adresse von Server B geschickt, sondern an die MAC-Adresse des eigenen Standardgateways (Default Gateway – die SVI-Schnittstelle auf dem L3-Switch).
3. **Änderung des L2-Headers:** Wenn der L3-Switch das Paket empfängt, ändert er die Quell- und Ziel-IP-Adressen im IP-Header (Layer 3) NIEMALS (der TTL-Wert wird lediglich um 1 verringert). Sobald das Paket den L3-Switch jedoch verlässt und in Richtung Server B in VLAN 20 weitergeleitet wird, wird der Ethernet-Header (Layer-2-Frame-Header) neu geschrieben: Die Quell-MAC-Adresse ist nun die VLAN-20-Schnittstelle des L3-Switches, die Ziel-MAC-Adresse die von Server B.
4. **Zustellung:** Das Paket wird innerhalb von VLAN 20 auf Layer-2-Ebene an Server B zugestellt.

---

## Fazit

Layer-2- und Layer-3-Technologien sollte man nicht als Konkurrenten, sondern als sich ergänzende Bausteine einer gemeinsamen Architektur betrachten. Switching hält den Traffic lokal und auf Hardwaregeschwindigkeit, während Routing diese lokalen Bereiche in kontrollierbare, sichere und skalierbare Strukturen überführt. In einer gut durchdachten Netzwerkarchitektur werden Layer-2-Grenzen so eng wie möglich gehalten und der Traffic so schnell wie möglich auf Layer-3-Ebene gehoben und dort verwaltet.
