---
date: "2018-10-15T00:00:00Z"
tags:
- tech
title: "Passwortreset bei importierten Usern in rocket.chat"
---

Wie einige vlt mitbekommen haben wurde HipChat vor kurzem von Slack [aufgekauft](https://www.theverge.com/2018/7/26/17619482/slack-hipchat-acquisition-stride-atlassian-partnership-microsoft-teams-competition) und wird anfang nächsten Jahres abgeschaltet. Da wir bisher in der Firma auf HipChat gesetzt haben, waren wir auf der Suche nach Ersatz.
Nach längerer Recherche sind wir dann bei [rocket.chat](https://rocket.chat/) gelandet. Es hat alle Features die wir brauchen (und noch VIELE mehr), OpenSource und es gibt eine gehostete Cloudversion davon.

Als Sahnehäubchen oben drauf haben sie sogar einen Importer für Hipchat Backups, der auch echt gut funktioniert. Dieser erspart einem das anlegen aller bestehenden Useraccounts und stellt die komplette Chat Historie in allen Räumen wieder her.

# Das Problem

Es werden zwar alle User aus dem Backup erfolgreich importiert und auch korrekt angelegt, jedoch haben die alle ja kein Passwort mit dem sie sich einloggen können. Auf meine [Nachfrage im Forum](https://forums.rocket.chat/t/set-password-for-lots-of-newly-imported-users/2238/2) kam der Hinweis ich soll das doch einfach über die [API](https://rocket.chat/docs/developer-guides/rest-api/) lösen. Gesagt getan, und kurz später stehen hier ein paar Zeilen Python die für alle neuen Useraccounts eine "Passwort zurücksetzen" Email triggern. Im Skript müssen lediglich die 3 globalen Variablen `BASE_URL`, `USERNAME` und `PASS` ganz oben angepasst werden.

Wenn ihr also gerade dabei seid eure alten Chat Backups in rocket.chat zu importieren (Sie unterstützen HipChat, Slack und selbstgebaute CSV) hilft euch mein Skript eventuell weiter.

{{< highlight python >}}
#! /usr/bin/env python3
import requests
from pprint import pprint
from sys import exit

BASE_URL = "https://companyname.rocket.chat"
USERNAME = "JohneDoe"
PASS = "hunter2"

def login():
    auth = None
    endpoint = "{}/api/v1/login".format(BASE_URL)
    payload = {"username": USERNAME,
               "password": PASS}
    request = requests.post(endpoint, json=payload)
    response = request.json()
    if request.status_code == 200 and response["status"] == "success":
        auth = {"X-User-Id": response["data"]["userId"],
                "X-Auth-Token": response["data"]["authToken"]}

    return auth


def get_users(auth):
    endpoint = "{}/api/v1/users.list".format(BASE_URL)

    request = requests.get(endpoint, headers=auth)
    response = request.json()
    if request.status_code == 200 and response["success"]:
        return response["users"]
    else:
        return None


def send_resetmail(users, auth):
    endpoint = "{}/api/v1/users.forgotPassword".format(BASE_URL)

    for u in users:
        email = u["emails"][0]["address"]
        print("Sending reset to {}...".format(email), end='')
        request = requests.post(endpoint, headers=auth, json={"email": email})
        response = request.json()

        if request.status_code == 200 and response["success"]:
            print("OK")
        else:
            print("FAIL")
            pprint(response)
            exit(1)

if __name__ == "__main__":
    auth = login()
    if auth:
        users = get_users(auth)
        if not users:
            print("Couldn't get a list of users")
            exit(1)

        new_users = [u for u in users if u.get(
            "emails", None) and not u["emails"][0]["verified"]]
        if len(new_users):
            print("Found {} new user. Will start sending password reset mails".format(len(new_users)))
            send_resetmail(users=new_users, auth=auth)
        else:
            print("Found 0 new user")
    else:
        print("Login failed")
        exit(1)
{{< / highlight >}}
