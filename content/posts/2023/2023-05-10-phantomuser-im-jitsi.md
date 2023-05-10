---
layout: post
date: "2023-05-10T00:00:00Z"
tags:
  - tech
title: "Phantomuser aus Jitsi Raum entfernen"
---


Nach einem Update von Jitsi auf unserem Jitsi Server, hingen ein paar alte Sessions permanent
in unserem Videochat die nur erstaunlich schwer weg zu bekommen waren.

<!--more-->

Die Tage hatte ich den Jitsi Server bei uns auf der Arbeit geupgradet während ich selbst noch in einem Call auf besagtem Server war. 
Während dem Upgrade wurde dann der Jitsi Service neu gestartet und ich bin aus dem Call geflogen. So weit so zu erwarten.
Als ich nach dem Upgrade jedoch wieder in den Raum joinen wollte, befanden sich dort zwei alte "Felix" User mit mir im Raum.
Das waren wohl alte Sessions die beim apprupten Terminieren von Jitsi während dem Upgrade nicht sauber terminiert wurden:

![Screenshot aus einem Jitsi call. Links oben ist meine Kamera und zwei weiter Felix user ohne Kamera](
    /img/2023/jitsi_call.png
    "Wat, wer ist das denn?")


Erste Reaktion war erstmal Jitsi nochmal "kontrolliert" mittels `systemctl` neuzustarten.  
**Resultat**: Kein Effekt...

Ja okay, dann schwerere Geschütze: Server mal komplett rebooten.  
**Resultat**: Kein Effekt...

Nächste Eskalationsstufe: Mal auf Mastodon fragen ob jemand der klugen Menschen im Fediverse weis wie ich das Problem löse.
Da sollten sich ja bestimmt auch einige Jitsi Admins finden lassen.  
**Resultat**: Es gibt jetzt dank [@stdevel](https://chaos.social/@stdevel) ein
tolles [Meme von mir](https://chaos.social/@fleaz/110316816350816496) :D


Da mir im IRC Channel von Jitsi auch niemand geantwortet hat, hab ich mich eben
mal selber auf dem Server auf die Suche gemacht. Irgendwo muss es ja "State"
geben, der noch die alten Sessions beinhaltet. Da ich wusste dass Jitsi intern auf XMPP basiert und daher hintendran
ein Prosody läuft, hab ich mir mal `prosodyctl` angeschaut. Das CLI tool um einen Prosody zu managen. Mittels `prosodyctl shell`
kommt man in eine aktive Adminshell und kann allerlei Commands ausführen:

```shell
$ prosodyctl shell


                     ____                \   /     _
                    |  _ \ _ __ ___  ___  _-_   __| |_   _
                    | |_) | '__/ _ \/ __|/ _ \ / _` | | | |
                    |  __/| | | (_) \__ \ |_| | (_| | |_| |
                    |_|   |_|  \___/|___/\___/ \__,_|\__, |
                    A study in simplicity            |___/


Welcome to the Prosody administration console. For a list of commands, type: help
You may find more help on using this console in our online documentation at 
https://prosody.im/doc/console

prosody> help
| Commands are divided into multiple sections. For help on a particular section, 
| type: help SECTION (for example, 'help c2s'). Sections are: 
| 
| Section  | Description                                                          
| c2s      | Commands to manage local client-to-server sessions                   
| s2s      | Commands to manage sessions between this server and others           
| http     | Commands to inspect HTTP services                                    
| module   | Commands to load/reload/unload modules/plugins                       
| host     | Commands to activate, deactivate and list virtual hosts              
| user     | Commands to create and delete users, and change their passwords      
| roles    | Show information about user roles                                    
| muc      | Commands to create, list and manage chat rooms                       
| stats    | Commands to show internal statistics                                 
| server   | Uptime, version, shutting down, etc.                                 
| port     | Commands to manage ports the server is listening on                  
| dns      | Commands to manage and inspect the internal DNS resolver             
| xmpp     | Commands for sending XMPP stanzas                                    
| debug    | Commands for debugging the server                                    
| config   | Reloading the configuration, etc.                                    
| columns  | Information about customizing session listings                       
| console  | Help regarding the console itself                                    
| 
```

Zuerst hab ich versucht über `user` die User zu listen und nur die Problemfälle zu entfernen, hier gab es jedoch nur 2 Einträge
für die aktuell tatsächlich aktiven User.

Des Rätsels Lösung liegt im `muc` Command. "MUC" steht hier für "multi-user chat", also das was Jitsi für einzelne Chaträume nutzt.
Mittels `muc:list()` und der Angabe eurer MUC Domain (Findet ihr in */etc/prosody/conf.d/<jitsidomain>).cfg.lua*):

```shell
prosody> muc:list("conference.euredomain.com")
| homeoffice@conference.euredomain.com
| OK: 1 rooms
```

Anschließend kann man den Raum mittels [:destroy()](https://prosody.im/doc/console#mucroom_roomdestroy) einfach terminieren, wobei Prosody vorher erst sauber alle User aus dem Channel wirft bevor der MUC geschlossen wird:

```shell
prosody>  muc:room("homeoffice@conference.euredomain.com"):destroy()
| Result: true
```

**Resultat**: Die Phantomuser sind weg und endlich wieder nur echte Menschen im Chat :D