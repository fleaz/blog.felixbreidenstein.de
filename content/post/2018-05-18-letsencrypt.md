---
date: "2018-05-18T00:00:00Z"
image: "img/letsencrypt-banner.png"
tags:
- tech
title: Ungültige Subdomains aus bestehendem Let's Encrypt Zertifikat entfernen
---

Von [Let's Encrypt](https://letsencrypt.org/) habt ihr ja hoffentlich alle schonmal gehört. Falls nicht hier die eigene Beschreibung von ihrer
Webseite:

{{< highlight text >}}
Let’s Encrypt is a free, automated, and open certificate authority (CA)
run for the public’s benefit. It is a service provided by
the Internet Security Research Group (ISRG).
{{< / highlight >}}

Oder runterbrochen auf das Wesentliche: Ihr bekommt da super kompfortabel und kostenlos SSL Zertifikate für eure Server die ohne Probleme in sämtlichen Browsern akzeptiert werden. Und das ganze auch noch mit super Tooling und fertig paketiert in den meisten Linux Betriebssystemen. Wenn ihr das jetzt unbedingt haben wollt empfehle ich euch für die eigenen Server den [Certbot](https://certbot.eff.org/) der EFF zu installieren. Für Menschen auf einem uberspace ist das ganze sogar noch einfacher, einfach mal ins [uberspace Wiki](https://wiki.uberspace.de/webserver:https#let_s-encrypt-zertifikate) schauen.

## Ungültig gewordene Domains

Ein Problem über das ich jedoch schon mehrmals gestolpter bin, und das man selbst wenn man weiß wonach man sucht, irgendwie nicht so schnell im Netz oder in der Doku findet ist folgendes: Ihr habt ein Zertifikat das auf mehrere Domains ausgestellt ist. Beispielsweiße für `foobar.com`, `www.foobar.com` und `static.foobar.com`. Das an sich ist ja kein Problem und kann mit mehreren `-d` Parametern beim Erstellen mit angegeben werden. Wenn man jedoch irgendwann bsbeispielsweiße den `static.` DNS Eintrag auf einen anderen Server umzieht fliegt der monatliche Cronjob beim nächsten Durchlauf auf die Nase, weil er natürlich versucht das Zertifikat so zu verlängern wie es auch erstellt wurde, aber für den umgezogenen DNS-Eintrag keine Validierung mehr bekommt (Das ganze passiert natürlich nur wenn ihr ACME mit HTTP-01 Validierung benutzt, und nicht mit DNS-01).

## Die Lösung

Am einfachsten behebt ihr das Problem folgendermaßen:
{{< highlight shell >}}
$ certbot renew --allow-subset-of-names
{{< / highlight >}}
Mit diesem Parameter erlaubt ihr Certbot das neue Zertifikat auch nur mit einer Teilmenge an Domains zu erstellen. In unserem Beispiel wären im neuen Zertifikat also nurnoch `foobar.com` und `www.foobar.com` mit drin. Problem gelöst.

Ihr werdet jetzt vielleicht sagen "Aber Felix, wieso schreib ich das dann nicht auch so in mein monatliches Skript und hab niewieder Probleme?". Das Problem dabei ist halt, dass ihr euch die Zertifikate automatisiert verbastelt unabhängig davon was der Grund war wieso certbot die Domain nicht verlängern konnte. Wenn ihr zum Beispiel einfach nur zu viel an eurem nginx vHost rumgebastelt habt und daher die HTTP-Challenge nicht ausgelesen konnte, fliegt die Domains aus dem Zertifikat. Und die BesucherInnen eurer Webseite bekommen dann ne dicke SSL-Warnung vom ihrem Browser präsentiert (die Sie wenn ihr [HSTS](https://de.wikipedia.org/wiki/HTTP_Strict_Transport_Security) aktiviert hattet, nichtmal wegklicken können), statt euren süßen Katzenbildern. Wäre doch schade, oder?
