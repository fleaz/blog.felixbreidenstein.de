---
title: "HAProxy - TCP-Proxy mit HTTP-Healthchecks"
date: 2025-10-05T00:00:00+02:00
tags:
  - tech
---

Sei es wegen High Availability, oder um eine große Anzahl von Requests zu bedienen – irgendwann kommt man an den Punkt, an dem man eine Anwendung über mehrere Server verteilen muss. Hier kommt ein Loadbalancer bzw. Reverse Proxy wie beispielsweise [HAProxy](https://www.haproxy.com/) ins Spiel: schnell, stabil, Open Source und mit tausenden Stellschrauben zum Feintunen.  
In diesem Post lernt ihr eine weitere dieser Stellschrauben kennen, um noch bessere Healthchecks für eure Upstream-Server zu bauen.

<!--more-->

Je nach Anwendung möchte man unterschiedliche Dinge mit dem Traffic am Loadbalancer machen. HAProxy kann hierfür auf verschiedenen Ebenen des [OSI-Modells](https://en.wikipedia.org/wiki/OSI_model) arbeiten, was man ihm über das Argument `mode` mitteilt.  
Mit `mode tcp` arbeitet HAProxy auf Layer 4 und schiebt einfach nur stumpf TCP-Streams zu euren Upstream-Servern – egal, ob mit oder ohne TLS und unabhängig von der eigentlichen Payload. Die Upstream-Server kümmern sich dabei auch um das Terminieren der TLS-Verbindungen, sodass euer HAProxy keine Zertifikate benötigt und nicht verstehen muss, welches Protokoll darin gesprochen wird. Das ist schnell, aber man hat auch weniger Features.  

Wenn ihr HAProxy jedoch mit `mode http` auf Layer 7 betreibt, sieht die Sache ganz anders aus: HAProxy **weiß** jetzt, dass es sich um HTTP-Traffic handelt, und kann protokollspezifische Änderungen vornehmen – etwa Header setzen oder Regeln anhand der URI anwenden. Das ist zwar etwas aufwendiger, bietet aber deutlich mehr Möglichkeiten.

Eines dieser Features ist ein umfangreicherer Healthcheck für eure Upstream-Server. HAProxy kann regelmäßig Requests an eure Backends schicken und prüfen, welche davon noch antworten. Kommt keine Antwort zurück, wird dieses Backend automatisch aus dem Pool genommen und bekommt keinen Traffic mehr.  
Für ein TCP-Backend ist dieser Check ein *SYN*-Request, der vom Backend mit einem *SYN/ACK* bestätigt werden muss. Es wird also nur geprüft, ob auf dem angegebenen Port eine TCP-Verbindung aufgebaut werden kann.

Wenn ihr jedoch ein HTTP-Backend habt, kann HAProxy auch die (heutzutage fast immer vorhandenen) Health-Endpunkte eurer Anwendung abfragen und somit deutlich zuverlässiger den Zustand des Servers bestimmen.

Hier zeigt sich nun ein Problem:  
Was ist, wenn man HAProxy als **TCP-Loadbalancer** nutzt, aber trotzdem gerne den besseren Healthcheck des HTTP-Modus hätte?

Unsere Rettung: [`track`](https://docs.haproxy.org/3.2/configuration.html#5.2-track)

Damit könnt ihr einem Backend-Server sagen, dass er den Healthcheck-Status eines anderen Servers übernehmen (tracken) soll.

Jetzt können wir einfach zwei Backends in der Konfiguration definieren:  
Das TCP-Backend nutzen wir für den eigentlichen Traffic in unserem Frontend, und das zweite Backend im HTTP-Modus verwenden wir nur für den besseren Healthcheck – die Server werden dabei über *track* referenziert.  
Das sieht dann in etwa so aus:

```haproxy {linenos=true}
frontend myApp
    bind *:6443
    default_backend myApp

backend myApp
    mode tcp
    server server-1 backend-1.example.com:6443 track monitoring/server-1
    server server-2 backend-2.example.com:6443 track monitoring/server-2
    server server-3 backend-3.example.com:6443 track monitoring/server-3

backend monitoring
    mode http
    option httpchk GET /readyz
    http-check expect status 200
    server server-1 backend-1.example.com:6443 check
    server server-2 backend-2.example.com:6443 check
    server server-3 backend-3.example.com:6443 check
```

Das *myApp*-Frontend zeigt auf das TCP-Backend, um dorthin den Traffic zu schicken (Zeile 3).  
Die drei Server in diesem TCP-Backend haben kein *check*-Argument gesetzt – sie führen also keine eigenen Healthchecks durch, sondern nutzen *track*, um auf die entsprechenden Server im Monitoring-Backend zu verweisen (Zeilen 7–9). Die Syntax für *track* lautet dabei: `backend-name/server-name`.

Im *monitoring*-Backend setzen wir `mode http` (Zeile 12) und konfigurieren unsere Optionen für den HTTP-Healthcheck (Zeilen 13–14).  
Dann listen wir noch einmal dieselben Server wie oben auf, diesmal aber mit dem *check*-Keyword.

Done!
HAProxy bleibt im TCP-Modus für den eigentlichen Traffic, aber das zweite Backend führt nun gescheite HTTP-Healthchecks für eure Upstream-Server durch.
