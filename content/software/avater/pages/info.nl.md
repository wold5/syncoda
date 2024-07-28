+++
title = "AVATeR Project info"
description = "Project info"
date = 2022-03-01T01:04:21+01:00
updated = 2024-03-18T00:00:00+01:00
weight = 0

[taxonomies]
tags = ["AVATeR", "C++", "Qt", "SQLite"]

[extra]
software = true
logo = "/images/logo-avater.png"
toc = true


+++
![](/images/avater-35px.png)
AVATeR staat voor Annotation Viewer and Tools for e-Readers (Annotatie weergave en Tools voor e-Readers).

Het programma ondersteunt PocketBook, Kobo en Sony e-readers.
- Annotaties kunnen interactief gelezen, gezocht en worden ge-exporteerd naar diverse formaten, direct vanaf een verbonden e-reader;
- Aantekeningen kunnen lokaal opgeslagen worden voor weergave na het loskoppelen van de e-reader; 
- diverse tools, voor het maken van back-ups en controle van databases, onder andere.

We beheren geen boeken of bibliotheken op de e-reader.

<!-- more -->

![AVATeR screenshot](/files/avater/screenshots/avater-screenshot-0.16-1.png)

## Informatie
Verdere informatie over AVATeR is volledig Engelstalig, en linkt u door naar de Engelstalige zijde van de website:
- [Engelstalige AVATeR homepage](/software/avater/pages/info)
	- [screenshots](/software/avater/pages/screenshots#screenshots)
	- [system requirements](/software/avater/pages/requirements/)
	- [e-reader compatibility](/software/avater/pages/compatibility/)
- [downloads/releases](/software/avater/releases/)

## Ondersteuning
- [online manual (English)](/software/avater/manuals/)
	- [Mobilereads topic](https://www.mobileread.com/forums/showthread.php?t=345428)</a>
	- e-mail naar support [at] syncoda.nl

### Programma structuur

AVATeR bestaat uit twee delen: de backend, waarvan de `USBScanner` tools omvat voor het vinden van USB apparaten, aan- en afkoppelingen en aankoppelingspunten (mountpoints); en de `ReaderManager` (Lezer beheerder), die Reader objecten beheert. Bij wijzigen signaleert deze de frontend UI applicatie, zoals de AVATeR UI; deze laatste omvat de annotatie weergave en gereedschappen.

![](/files/avater/programdiagram.svg)

De USBScanner zou omgezet kunnen worden naar C; het interacteert direct met WIN32/udev APIs, al zou `libusb` hiervoor ook gebruikt kunnen worden in de toekomst. De ReaderManager is geschreven in C++ (in een C-achtige stijl); en bevat nog diverse Qt afhankelijkheden voor o.a. SQL, voorkeuren en debug logging. De UI bouwt op C++ en Qt. Betreft Qt is de applicatie compatibel met Qt 5.12+ en 6.7 en later.

De programma bronnen zijn momenteel gesloten; de intentie is deze openbaar te maken. Het idee is om eerst de lagere delen te openen, beginnend met de USBScanner. Een liberale licentie heeft de voorkeur.

### Geschiedenis
AVATeR begon als de PocketBook Tools plugin voor Calibre; een eerste zelfstandig draaiende versie had een klein scherm met 6 knoppen voor gereedschappen. De originele annotatie HTML export tool (SQL gebaseerd) werd vervangen door een interactieve weergave, en het een leidde tot het ander.

## Donaties

U wordt aangemoedigd om te doneren, als dit voor u (financieel) mogelijk is.

- [Donate using PayPal](https://www.paypal.com/donate/?hosted_button_id=4UATTKGGJ9V68)

---

Voor downloads [zie de (Engelstalige) release pagina](/software/avater/releases/).

