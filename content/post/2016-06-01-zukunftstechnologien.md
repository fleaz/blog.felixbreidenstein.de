---
date: "2016-06-01T00:00:00Z"
image: "img/matrix.png"
tags:
- news
- tech
title: The future is now!
---

Ist ja immerhin schon 2016. Da kann eins jetzt auch langsam mal von Zukunft sprechen.

Und weil ja jetzt "Zukunft" ist, müssen auch Zukunftstechnologien her! Also hab ich ein bisschen gebastelt und die Technik hinter diesem
Blog ein bisschen verbessert.

1) IPv6
========
Eigentlich keine Neuerung weil dieser Blog schon von Begin an, DualStack über IPv4 und IPv6 augeliefert wird. Aber da die Verbreitung von v6 leider echt noch zu wünschen übrig lässt, kann das schon noch als Zukunftstechnologien gezählt werden. IPv6 gibs ja auch erst seit 1998. Das dauert halt bis das überall implementiert ist...

2) X509 Zertifikat
=================
Auch Transportverschlüsselung in Form von TLS hat dieser Blog schon von Anfang an gehabt. Bis vorhin lief das aber noch mit einem Zertifikat von StartCom. StartCom ist eine Firma aus Israel die sich unter anderem als CA (Certificate authority) einen großen Namen gemacht hat, da ihre SSL Zertifikate kostenlos sind. Die Dinger sind 1 Jahr gültig und danach muss eins sich wieder in dem (unglaublich schlechten) Webinterface einloggen und ein neues bauen.
Seit April diesen Jahres gibt es jedoch einen neuen Player im Spiel: [Let's Encrypt](https://letsencrypt.org/). Eine CA die von Mozilla, der EFF und der University of Michigan gegründet wurde und sich zum Ziel gesetzt hat, sehr einfach und automatisiert an alle kostenlose TLS Zertifikate zu verteilen.
Für das "automatisch" haben sie ACME entwickelt. Ein Protokoll das euer Server mit den Servern von LE reden kann um nachzuweisen dass eine Domain wirklich euch gehört. Dafür wird eine kleine Datei angelegt die dann über die zu validierende Domain angefragt wird. Gehört die Domain euch kann der ACME Server von außen auf die Datei zugreifen und "glaubt" euch damit dass ihr InhaberIn der Domain seid. Wenige Sekunden später landet in "/etc/letsencrypt/live" ein Zertifikat für die angefragte Domain. Um sich das Revoken bei gestohlenen Zertifikaten zu sparen, setzt LE auf extrem kurze Gültigkeitsdauern. Aktuell sind es 3 Monate, aber es waren am Anfang auch schon wenige Tage im Gespräch. Was damit zwangsläufig einhergeht, ist die benötigte Automatisierung. Und dass ist auch von LE genau so gewollt. Spätestens wenn irgendwann die Zertifikate bspw. nurnoch 72h gültig sind, MUSS der Erneuerungsprozess automatisiert werden. Oder wollt ihr euch wirklich alle 3 Tage auf allen euren Servern einloggen und neue Zertifikate beantragen? Automatisieren kann eins das z.b. ganz einfach mit einem cron-job oder systemd-Timer. Als Beispiel für den geringen Aufwand der betrieben werden muss um ein Zertifikat zu beantragen, hier eine Liste der Befehle die ich benutzt habe um das Zertifikat für diesen Blog zu generieren:

{{< highlight bash >}}
$ sudo apt-get install certbot -t jessie-backports
$ certbot certonly --webroot -w /var/www/www.felixbreidenstein.de/htdocs -d www.felixbreidenstein.de -d felixbreidenstein.de
{{< / highlight >}}

Ja, das wars schon. Jetzt nurnoch die Zertifikate in die Config des Webservers einbauen. Und um ALLE LE Zertifikate die auf dem System erstellt wurden zu erneuern, führt einfach folgenden Befehl aus:

{{< highlight bash >}}
$ certbot renew
{{< / highlight >}}

TLS Zertifikate generieren hat also das "Baby einen Lutscher klauen" Level erreicht.

3) HTTP/2
=========
Kommen wir nun zum eigenlichen Höhepunkt der Neuerungen die auch erst seit heute Mittag funktioniert: HTTP/2.
Als Version 2.0 des HTTP Protokolls mit dem Webseiten ausgeliefert werden bring es einige ziemlich coole Neuerungen mit. Die aktuelle Version 1.1 ist ursprünglich von 1997 (danach noch mehrmals überarbeitet) und passt daher nichtmehr so ganz zum Internet wie wir es heute kennen. Daher hat sich die "httpbis" Arbeitsgruppe der [IETF](https://www.ietf.org/) hingesetzt und Version 2.0 entwickelt. Im Mai 2015 war es dann soweit und HTTP/2 erblickte in Form von [RFC 7540](https://tools.ietf.org/html/rfc7540) das Licht der Welt. Die meisten Neuerungen sind Performanceoptimierungen um aktuelle Webseiten, welche ruhig schonmal 2-stellige MB Zahlen* auf die Waage bringen, schneller zu laden. So kann der Server mit HTTP/2 schon Ressourcen an den Browser schicken bevor diese angefragt wurden und mehrere Requests können durch Multiplexing parallel übertragen werden.
In den [dotdeb](https://www.dotdeb.org/) Repositoys, aus denen ich meinen nginx Webserver installiere, befindet sich seit gestern die Version 1.10 des selbigen.
Mit dieser Version kann ich nun folgendes in den ssl server-Block meiner Config schreiben:
{{< highlight bash >}}
listen 5.199.142.236:443 ssl http2;
listen [2001:4ba0:fffa:1dc::1]:443 ssl http2;
{{< / highlight >}}
und schon wird dieser Text mit HTTP/2 zu euch übertragen, solange ihr einen halbwegs aktuellen Browser habt.
Weiterer cooler Nebeneffekt von HTTP/2: Das Protokoll ist zwar sowohl für HTTP als auch für HTTPS spezifiziert, die großen Browserentwickler haben sich aber gedacht: "Nope, wir machen HTTP/2 nur über TLS". Mit diesem Move, und in Verbindung mit Let's Encrypt, werden wir hoffentlich bald alle HTTP-only Seiten los.

*) Hierzu noch eine lustige Anekdote: Ronan Cremin hat sich anfang 2016 mal hingesetzt und die durchschnittliche Größe von heutigen Webseiten angeschaut. Das Ergebniss: Eie heutige Webseite ist genauso groß wie das original DOOM Spiel. [Quelle](https://mobiforge.com/research-analysis/the-web-is-doom)



(Headerimage by  by Jamie Zawinski [Link](https://commons.wikimedia.org/wiki/File:The.Matrix.glmatrix.2.png))
