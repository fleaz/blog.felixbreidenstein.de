---
layout: post
date: "2023-02-07T00:00:00Z"
tags:
  - tech
  - smarthome
title: "Firmwareupdate für Conbee II Zigbee Stick"
---

Und wieder einmal zeigt sich: Updaten hilft :D

<!--more-->

Vielen kennen vermutlich den weit verbreiteten [Conbee II](https://phoscon.de/en/conbee2) der deutschen Firma "dresden elektronik". Mit 30€ relativ
günstig und in vielen Shops immer auf Lager (im vergleich zu anderen beliebten Zigbee Sticks), ist er oft die erste
Wahl um herstellerunabhängig Zigbee reden zu können und Geräte bspw. in ein HomeAssistant zu integrieren. So auch bei
mir.

Zwar lief immer alles Problemlos, aber ich habe in meinem dmesg gesehen dass der Stick im Minutentakt die USB Verbindung
verloren hat und neu connected ist. Das sah dann so aus:

```
...
[ 3590.828132] cdc_acm 1-5:1.0: ttyACM0: USB ACM device
[ 7191.730866] usb 1-5: USB disconnect, device number 12
[ 7191.981063] usb 1-5: new full-speed USB device number 13 using xhci_hcd
[ 7192.109090] usb 1-5: New USB device found, idVendor=1cf1, idProduct=0030, bcdDevice= 1.00
[ 7192.109102] usb 1-5: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[ 7192.109106] usb 1-5: Product: ConBee II
[ 7192.109110] usb 1-5: Manufacturer: dresden elektronik ingenieurtechnik GmbH
[ 7192.109113] usb 1-5: SerialNumber: DE2463625
[ 7192.112438] cdc_acm 1-5:1.0: ttyACM0: USB ACM device
[ 7195.552543] usb 1-5: USB disconnect, device number 13
[ 7195.855892] usb 1-5: new full-speed USB device number 14 using xhci_hcd
[ 7201.403776] usb 1-5: unable to read config index 0 descriptor/all
[ 7201.403795] usb 1-5: can't read configurations, error -110
[ 7201.517654] usb 1-5: new full-speed USB device number 15 using xhci_hcd
[ 7207.035515] usb 1-5: unable to read config index 0 descriptor/all
[ 7207.035533] usb 1-5: can't read configurations, error -110
[ 7207.035641] usb usb1-port5: attempt power cycle
[ 7207.415402] usb 1-5: new full-speed USB device number 16 using xhci_hcd
[ 7211.586774] usb 1-5: unable to read config index 0 descriptor/all
[ 7211.586792] usb 1-5: can't read configurations, error -71
[ 7211.903219] usb 1-5: new full-speed USB device number 17 using xhci_hcd
[ 7211.917330] usb 1-5: New USB device found, idVendor=1cf1, idProduct=0030, bcdDevice= 1.00
[ 7211.917341] usb 1-5: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[ 7211.917345] usb 1-5: Product: ConBee II
[ 7211.917348] usb 1-5: Manufacturer: dresden elektronik ingenieurtechnik GmbH
[ 7211.917351] usb 1-5: SerialNumber: DE2463625
[ 7211.919216] cdc_acm 1-5:1.0: ttyACM0: USB ACM device
...
```

Kurz den Fehler in die Suchmaschine der Wahl geworfen und es zeigte sich, dass ich mit dem Problem nicht alleine war. In einem
der Github Issues von deCONZ (die eigene Management Software von dresden elektronik für Zigbee Netzwerke), wurde
empfohlen ein Firmwareupdate des Sticks zu machen, was mit der genannten deCONZ Software auch kein Problem ist. Statt
sich das komplette deCONZ Tool zu installieren, kann man jedoch auch separat einfach nur das Flash Utility
installieren welches bei  deCONZ mit dabei ist. Das hört auf den Namen *GCFFlasher* und liegt OpenSource auf
[Github](https://github.com/dresden-elektronik/gcfflasher). Einfach das Repo clonen und der sehr kurzen
Installationsanleitung aus der README folgen, schon habt ihr die Binary im gleichen Ordner liegen (Oder ihr nehmt das
[fertige Paket für NixOS](https://github.com/NixOS/nixpkgs/pull/215149) was ich gerade schnell paketiert habe
#NixOSUltras ). Dann sucht ihr euch noch [hier](https://deconz.dresden-elektronik.de/deconz-firmware/) die aktuellste
Firmware für euren Stick aus (Es gibt hier die Firmware alle Conbee und Raspbee Produkte) und ladet sie herunter.
Anschließend nurnoch folgenden Befehl (mit angepassten Dateinamen) ausführen:

```bash
./GCFFlasher -t 60 -d /dev/ttyACM0 -f deCONZ_ConBeeII_0x26780700.bin.GCF
```

Nach wenigen Sekunden sollte der Updatevorgang erledigt sein und ihr habt euren Stick auf die aktuellste Firmwareversion
gebracht :)

Dadurch sind jedefalls für mich, sämtliche USB Resetprobleme verschwunden.
