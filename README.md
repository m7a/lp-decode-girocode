---
section: 32
x-masysma-name: decode_girocode
title: Girocode-Dekodierungsskripte
date: 2022/05/13 19:06:28
lang: de-DE
author: ["Linux-Fan, Ma_Sys.ma (Ma_Sys.ma@web.de)"]
keywords: ["girocode", "qrcode", "zbar-tools"]
x-masysma-version: 1.0.0
x-masysma-website: https://masysma.lima-city.de/32/decode_girocode.xhtml
x-masysma-repository: https://www.github.com/m7a/lp-decode-girocode
x-masysma-owned: 1
x-masysma-copyright: |
  Copyright (c) 2018, 2022 Ma_Sys.ma.
  For further info send an e-mail to Ma_Sys.ma@web.de.
---
Einführung
==========

Dieses Repository enthält zwei Skripte, die zum Dekodieren von Rechnungen mit
Girocodes gedacht sind.

Der Girocode ist ein QR-Code, der die Bezahldaten (Empfänger, Verwendungszweck,
Betrag etc.) codiert und in der Wikipedia gut beschrieben ist:
<https://de.wikipedia.org/wiki/EPC-QR-Code>.

Der Code ist vornehmlich gedacht, um mit Smartphone-Kameras gescannt und
dann in einer Online-Bankinganwendung auf dem Smartphone verarbeitet zu werden.

Wenn man jedoch die Überweisungen lieber am „richtigen Rechner” durchführt, so
akzeptieren die meisten Online-Bankinganwendungen keine direkte Verwendung des
Girocode.

Die Skripte aus diesem Repository bieten eine einfache Lösung für diese Lücke
und decken zwei Anwendungsfälle ab.

Anwendungsfall: Girocode mit Barcode-Scanner einscannen
=======================================================

Wer über einen QR-Code-fähigen Barcode-Scanner zum Anschluss an den Rechner
-- bspw. per USB, getestet mit [Datalogic QuickScan I QD2430](https://www.conrad.de/de/p/datalogic-quickscan-i-qd2430-barcode-scanner-kabelgebunden-1d-2d-imager-schwarz-hand-scanner-usb-1308800.html) --
verfügt, kann das Skript folgendermaßen aufrufen:

	./decode_girocode

Anschließend betätigt man den Scanner und die Überweisungsdaten werden auf
der Konsole dekodiert ausgegeben. Mit etwas Übung ist auch der Inhalt des
QR-Codes direkt lesbar, aber mit der skriptbasierten Decodierung kommt es nicht
zu Verwechslungen, welche Daten in welches Feld bei der Überweisung einzutragen
sind.

Ein Test mit der Beispielüberweisung von der oben verlinkten Wikipedia-Seite
ergibt bspw. folgendes:

~~~
$ ./decode_girocode
BCD
001
1
SCT
BFSWDE33BER
Wikimedia Foerdergesellschaft
DE33100205000001194700
EUR123.45


Spende fuer Wikipedia

--------------------------------------------------------------------------------

Zahlungsempfänger Wikimedia Foerdergesellschaft
Ziel-IBAN         DE33100205000001194700
Betrag            EUR123.45
Verwendungszweck  Spende fuer Wikipedia

--------------------------------------------------------------------------------
~~~

Anwendungsfall: Girocode aus PDF-Rechnung nutzen
================================================

Mit dem Tool `zbarimg` können Bilddateien auf enthaltene Barcodes untersucht
und das Ergebnis auf der Standardausgabe ausgegeben werden. Dieses lässt sich
in Verbindung mit GhostScript zum Rendern von PDFs als Bitmaps hervorragend
nutzen, um eine als PDF empfangene Rechnung zu dekodieren.

	./decode_girocode_pdf DATEINAME.pdf

Auch hier kann das Wikipedia-Beispiel zum Testen genutzt werden, indem man den
QR-Code mit `wkhtmltopdf` von der Webseite in ein PDF lädt:

~~~
$ wkhtmltopdf https://de.wikipedia.org/wiki/Datei:Index2wikipedia.png test.pdf
Loading page (1/2)
Printing pages (2/2)
Done
$ ./decode_girocode_pdf test.pdf
GPL Ghostscript 9.53.3 (2020-10-01)
Copyright (C) 2020 Artifex Software, Inc.  All rights reserved.
This software is supplied under the GNU AGPLv3 and comes with NO WARRANTY:
see the file COPYING for details.
Processing pages 1 through 1.
Page 1
--------------------------------------------------------------------------------

Zahlungsempfänger Wikimedia Foerdergesellschaft
Ziel-IBAN         DE33100205000001194700
Betrag            EUR123.45
Verwendungszweck  Spende fuer Wikipedia

--------------------------------------------------------------------------------
~~~

Will man stattdessen das ImageMagick-Backend verwenden, so kann man die
entsprechenden Befehle auch direkt auf dem Terminal eingeben und zuerst ein
PNG erzeugen, dessen enthaltener QR-Code anschließend mit `zbarimg` dekodiert
und mit `decode_girocode` weiterverarbeitet wird:

	convert -density 600 test.pdf test.png
	zbarimg "-S*.enable=0" -Sqrcode.enable=1 test.png | ./decode_girocode

Hierbei sind die Parameter `-S` für `zbarimg` entscheidend, sodass nur der
QR-Code und keine sonstigen möglicherweise enthaltenen Barcodes verarbeitet
werden. Darüber hinaus stellt `-density 600` beim `convert`-Aufruf sicher, dass
der QR-Code in hinreichend hoher Auflösung gerendert wird.

Einschränkungen
===============

Die Überweisungsdaten sind unbedingt zwischen dem TAN-Generator/2. Faktor mit
dem Original-PDF bzw. Zettel abzugleichen, um zu vermeiden, dass falsche
Datenverarbeitung durch das Skript zu finanziellen Schäden führt.

Zur Verarbeitung von Mehrseitigen PDFs, bei denen der QR-Code nicht auf der
ersten Seite des PDFs steht, muss zuerst die relevante Seite extrahiert werden,
bspw. mit dem Programm `pdfjam`.

Auch wenn es aus der Lizenz klar sein sollte: Es wird ausdrücklich keine
Garantie für die Funktionsfähigkeit der Skripte übernommen.

Hilfreiche Ressourcen
=====================

 * Girocode-Spezifikation (aka. „EPC-QR-Code”):
   <https://www.europeanpaymentscouncil.eu/document-library/guidance-documents/quick-response-code-guidelines-enable-data-capture-initiation>
 * [gs(1)](https://manpages.debian.org/bullseye/ghostscript/gs.1.en.html)
 * [convert(1)](https://manpages.debian.org/bullseye/graphicsmagick-imagemagick-compat/convert.1.en.html)
 * [zbarimg(1)](https://manpages.debian.org/bullseye/zbar-tools/zbarimg.1.en.html)
