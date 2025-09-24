---
title: "Ein Haus bauen, einen Baum pflanzen, einen k8s Switcher schreiben..."
date: 2025-08-27T00:00:00+02:00
tags:
  - tech
---

Manchmal muss man halt das Rad auch mal neu erfinden. Und dann schreibt man ein Skript was schon viele Menschen vor
einem geschrieben haben.

<!--more-->

Seit meinem Teamwechsel vor ein paar Monaten, bin ich ja jetzt auch YAML Engineer und turne den ganzen Tag in Kubernetes
Clustern herum. Wenn man das beruflich macht, hat man da dann auch meist mehr als eins von, und dann merkt man recht schnell
dass man irgend was braucht um zwischen den Clustern zu switchen.

Defaultmäßig steht die Configuration für den lokalen k8s Client in der Datei `~.kube/config`. In der Datei
befinden sich eine Liste von Clustern mit ihrem Hostname und eine Liste von Accounts. Welcher Account für welches
Cluster ist, wird per Context definiert, davon steht auch eien Liste von in dieser Datei. Ein Context verknüpft immer
einen Account aus der Datei mit einem Cluster. Wenn man zur Authentication bspw OIDC nutzt, kann ja das gleiche Auth
Objekt für mehrere Cluster beuntzt werden.

Um jetzt mit kubectl an einem anderen Cluster zu arbeiten braucht man bspw folgendes Kommando:
```shell
$ kubectl config use-context prod-fra
```

Um jetzt in diesem Cluster noch in den richtigen Namespace zu springen tippt man noch schnell
```shell
$ kubectl config set-context --current --namespace=monitoring
```
hinterher. Man sieht, das geht echt nicht schnell von der Hand, und wenn man jetzt nen halbes Dutzend Cluster mit jeweils 10-20
Namespaces hat, ist das kein Spaß mehr.

Das dachten sich auch andere, weswegen es da einige Tools auf dem Markt gibt die einem ein schnelleres Springen zwischen
Contexten und Namespaces erlauben. Bekannte Beispiele sind hier bspw. [kubens](https://github.com/roubles/kubens/) oder
[kubectx](https://github.com/ahmetb/kubectx).

Ich hab ne Hand voll verschiedener Lösungen ausprobiert und bin am Ende dann doch bei selber basteln gelandet.
Rausgekommen ist **ktx** welches in meinem Forgejo unter [git.rainbownerds.de/fleaz/ktx](https://git.rainbownerds.de/fleaz/ktx) zu finden ist.

Meine zwei wichtigsten Anforderungen an so ein Tool waren:
  * Wenn ich ein neues Terminal aufmache, hab ich dort immer die zuletzt benutze Umgebung
  * Wenn ich in einem Terminal die Umgebung wechsle (egal ob Namepace oder ganzes Cluster), hat das keinen Einfluss auf
      andere bereits geöffnete Terminals

Grund hierfür ist, dass man oft in der Situation ist dass man mehrere Terminals nebeneinander offen hat um glechzeitig
in verschiedene Cluster zu schauen. Nur weil ich in dem anderen Terminal gerade ins Prod Cluster gesprungen bin, soll
mein dev Cluster in dem ich gerade was teste, bitte in der dev bleiben. Das gibt sonst ein schlimmes erwachen :D

Wenn man sich nun mein Tool "installiert" (Ein großes Wort für 'schreib eine include Zeile in deine Shell config'), hat
man die Befehle **ktx** und **kn**. Beide können einfach ohne Argumente aufgerufen werden und wechseln einmal den
gesammten Context, oder nur den Namespace im aktuellen Cluster. Beides wurde mit einer interaktiven Liste mittels fzf
gebaut, dadurch kann man schnell durch fuzzymatching Dinge auswählen und hat IMHO ein schönes UI.

Die Unabhängigkeit zwischen den Terminals entsteht dadurch dass ich mittels kubectl, nach dem Wechseln von Cluster oder
Namespace, den aktuellen Kontext in ein seperates File unter `~/.kube/ktx` schreibe und dann die Umgebungsvariable
`KUBECONFIG` auf diese Datei setze. Ist diese Datei gesetzt, wird diese Datei als k8s Config geladen und nicht die
Defaultconfig. Danach wird dann ein *current* Symlink aktualisiet der immer auf das neueste File zeigt, der von jedem
neuen Terminal geladen wird.

Und nach mehrmonatigem Testen im Vollzeitjob, kann ich sagen dass das für mich sehr gute 65 Zeilen Bash sind ;)
