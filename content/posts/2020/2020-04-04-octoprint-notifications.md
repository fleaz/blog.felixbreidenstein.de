---
layout: post
date: "2020-04-04T00:00:00Z"
image:
  feature: "img/octoprint_notification.jpg"
tags:
  - tech
  - smarthome
title: Wissen wann der 3D-Drucker fertig ist
---

Wenn man einen 3D-Drucker hat druckt man zwangsläufig irgenwann auch mal größere Dinge die viele Stunden brauchen.
Spätestens bei solchen Teilen oder wenn der Drucker in einem anderen Raum/Stockwerk steht, sitzt man nichtmehr
durchgehend daneben sondern macht währenddessen etwas anderes (*Siehe Warnhinweise ganz unten!*). Wenn man nun wie ich
bereits Homeassistant und Octoprint im Einsatz hat, kann man sich darüber realtiv leicht eine Pushnotification schicken lassen
sobald der Drucker fertig ist.

<!--more-->

## Octoprint
[Octoprint](https://octoprint.org/) ist ein großartiges Stück Software welches man bspw auf einem
RaspberryPi installieren kann der über USB an einem 3D Drucker angeschlossen ist. Damit kann man dann super
kompfortable Druckauftäge hochladen und starten, die Temperaturen überwachen oder sich mit einer Webcam den aktuellen
Fortschirtt anschauen. Falls ihr einen 3D-Drucker habt aber noch kein Octoprint benutzt, lege ich euch das sehr ans
Herzen. Ein Raspberry ist nicht sehr teuer (im Vergleich zu einem 3D Drucker ;) ) und es wird euer Leben so viel
kompfortabler machen weil man nichtmehr mit der SD-Karte zwischen PC und Drucker hin und her laufen muss.

## Homeassistant
[Home Assistant](https://www.home-assistant.io/) ist eine open-source smarthome/homeautomation Lösung. Damit steuern
wir hier bei uns in der WG bspw alle Steckdosen,die Tradfri Lampen, überwachen Temperatur/Luftfeuchtigkeit in den
Räumen, schauen ob die Waschmaschine schon fertig ist und bspw eben auch den Status meines 3D-Druckers.

## Telegram
Ja, ich weis Telegram ist böse, closed-source etc. pp. aber war halt jetzt für meinen Zweck auf die schnelle das
einfachste um aus Homeassistant Notifications an mein Handy zu schicken. Und unsere WG Chatgruppe ist auch auf Telgram (Kleinster gemeinsammer Nenner und so), daher hab ich die App also eh schon auf dem Handy.

## And now kiss!

Die drei genannten Komponenten muss man nun miteinander kombinieren. Auf die Installation von Octoprint und HomeAssistant werd ich hier jetzt nicht eingehen, dazu gibt es schon genug gute Anleitungen im Netz.

### Octoprint Modul

Zuerst schnappt ihr euch eure `configuration.yaml` von HomeAssistant und baut das Octoprint-Modul ein. Den API-Key bekommt ihr im Webinterface von Octoprint in den Einstellungen (Schraubenschlüssel rechts oben in der Navbar) unter "API".
```yaml
octoprint:
  host: http://octoprint.wg.ibg10.de
  api_key: !secret octoprint_key
  bed: True
  number_of_tools: 1
```

Nach einem Neustart von Hass sollten nun eine Liste von neuen Sensoren aufgetaucht sein:
![Liste der Sensoren in Homeassistant](/img/octoprint_sensoren.png)

### Telegram integration
Hierfür müsst ihr bei Telegram erst einen Bot anlegen damit ihr über die API von ihnen Nachrichten verschicken könnt. Wie das funktioniert ist ganz gut im HomeAssistant Wiki erklärt: [Klick mich](https://www.home-assistant.io/integrations/telegram/)

Anschließend könnt ihr wieder in der `configuration.yaml` zwei weitere Blöcke hinzufügen. Einen für den Bot an sich und einen für die "Emfpängergruppe" suzusagen. In meinem Beispiel ist das `telegram_felix`:

```yaml
telegram_bot:
  - platform: polling
    api_key: !secret telegram_key
    allowed_chat_ids:
      - 12345678

notify:
  - name: telegram_felix
    platform: telegram
    chat_id: 12345678
```

### Optional: Steckdosen
Bei mir hängt sowohl der Drucker als auch die LED Beleuchtung dafür ein zwei
[Funksteckdosen](https://www.obi.de/hausfunksteuerung/wifi-stecker-schuko/p/2291706) die ich auf
[Tasmota](https://github.com/arendst/Tasmota) umgeflasht habe. Somit kann ich beides über Homeassistant an/aus schalten. Die Konfiguration dafür sieht bspw so aus:
```yaml
switch:
  - platform: mqtt
    name: "3D Drucker LEDs"
    command_topic: "cmnd/A00E8C/POWER"
    state_topic: "stat/A00E8C/POWER"
    payload_on: "ON"
    payload_off: "OFF"

  - platform: mqtt
    name: "3D Drucker"
    command_topic: "cmnd/128524/POWER"
    state_topic: "stat/128524/POWER"
    payload_on: "ON"
    payload_off: "OFF"
```

### Automation
*Automations* in Homeassistant sind Aktionen die automatisch ausgeführt werden wenn ein bestimmtes Event eintritt
und/oder eine Bedingung erfüllt ist. In unserem Falle wäre das also "Der Drucker wechselt von 'Printing' auf
'Operational'" das Trigger-Event und "Sende eine Pushnotification und schalte die beiden Steckdosen aus" die Aktion.
Mit einer aktuellen Version von Homeassistant kann man diese Automations sehr einfach über das Webinterface erstellen. Geht dazu auf
`/config/automation` in eurem Homeassistant und klickt unten rechts auf das Plus. Wenn ihr alles konfiguriert habt, sollte es in etwa wie in folgendem Screenshot aussehen:
![Screenshot aus Homeassistant](/img/homeassistant_automation.png)

Jetzt nurnoch unten rechts auf speichern klicken und schon wisst ihr immer bescheid wann euer Druckauftrag fertig ist :)

Einziges Manko dieser Lösung ist leider dass man auch eine Notification erhällt wenn man den Druck abbricht, da der Status hier dann auch von "Printing" nach "Operational" wechselt. Also Lösung dafür könnte man eventuell noch den Sensor **sensor.octoprint_time_remaining** als Bedingung in die Automation hinzufügen und nur weitermachen wenn die Restzeit auf 0 steht.


# Warnhinweis
3D-Drucker sind technische Geräte mit sehr viel Strom und stellenweiße auch sehr hohen Temperaturen! Vorallem wenn
der Drucker noch neu ist oder ihr Änderungen daran vorgenommen habt solltet ihr ihn auf keinen Fall stundenlang
unbeobachtet laufen lassen. Außerdem solltet ihr bei günstigen Druckern darauf achten auf jeden Fall die Firmware neu
zu flashen und dort die "Therman Runaway Protection" zu aktivieern. Dadurch erkennt der Drucker wenn einer der
Temperaturfühler defekt ist und stoppt die Maschine statt unendlich weiterzuheizen und auf die Zieltemperatur zu kommen. Ohne
TRP [brennt euch im besten Fall nur der Drucker ab](https://www.thissmarthouse.net/dont-burn-your-house-down-3d-printing-a-cautionary-tale/), im worst case halt
euer ganzes Haus! Ein Rauchmelder in dem Raum wo der Drucker läuft kann also jedenfalls nicht schaden.
