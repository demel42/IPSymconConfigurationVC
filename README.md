# IPSymconConfigVC

[![IPS-Version](https://img.shields.io/badge/Symcon_Version-5.0-red.svg)](https://www.symcon.de/service/dokumentation/entwicklerbereich/sdk-tools/sdk-php/)
![Module-Version](https://img.shields.io/badge/Modul_Version-1.0-blue.svg)
![Code](https://img.shields.io/badge/Code-PHP-blue.svg)
[![License](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-green.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)
[![StyleCI](https://github.styleci.io/repos/126683101/shield?branch=master)](https://github.styleci.io/repos/146979798)

## Dokumentation

**Inhaltsverzeichnis**

1. [Funktionsumfang](#1-funktionsumfang)
2. [Voraussetzungen](#2-voraussetzungen)
3. [Installation](#3-installation)
4. [Funktionsreferenz](#4-funktionsreferenz)
5. [Konfiguration](#5-konfiguration)
6. [Anhang](#6-anhang)
7. [Versions-Historie](#7-versions-historie)

## 1. Funktionsumfang

Es werden verschiedene Dateien und Daten aus dem IP-Symcon-Verzeichnis in Git gesichert und versioniert.

Die Daten sind
1. Scripte<br>
die Scripte aus dem Scriptverzeichnis werden, falls geändert, übernommen. Neue Dateien werden hinzugefügt, zwischenzeitlich gelöschte Dateien werden auch im Repository gelöscht.
2. Module<br>
Die installierten Module werden untersucht. Es wird pro Modul eine Datei _<Modul-Verzeichnis>.json_ angelegt, die die  _Url_, den _Branch_ und den zeitpunkt der letzten Modifikation einer Datei aus dem Modul-Verzeichnis enthält.
Die Datei ändert sich somit nur, wenn sich auch in dem Modul etwas geändet hat.
Zusätzlich gibt es die Möglichkeit, das Modul als solches auch als Zip-Archiv zu sichern. Aufgrund der Größe mancher Module funtioniert das aber nicht unbedingt mit öffentlichen Git-Servern (_GitHup_ hat z.B. zur Zeit eine Grenze von 100 MB pro Datei).
3. Setting<br>
Die Settigs wird als Ganzes gesichert.
Da sich der Inhat von _settings.json_ sehr dynamisch ändert und ein vorher/nachher-Vergleich sehr unübersichtlich ist, wird noch folgendes gemacht:
a) aus _settings.json_ werden alle Bereiche außer _objects_ als eigene _.json_-Datei gesichert (also _profiles.json_ und _options.json_). Diese Dateien werden nur geschrieben, wenn sich am Inhalt etwas geändert hat.
b) Objekte<br>
alle Objekte werden aus dem IPS heraus (also nicht aus der _settings.json_) gesichert, dabei wir pro Objekt eine Datei angelegt. Diese Datei enthält die json-Strukturen die von den jeweiligen IPS-Aufrufen (_IPS_GetObject()_ sowie _IPS_GetInstance()_ etc) geliefert werden. Um nur relevante Änderungen zu sehen werden eventuelle Zeitstempel oder die Werte der Variablen entfernt. Da die Datei nur geschrieben wird, wenn sich etwas geändert hat.
4. README.md
In dieser Datei wird ein Protokoll der Änderungen des letzten Abgleichs dargetsellt. Man sieht als sehr schnell, an welcher Stelle der Konfiguration seit dem letzten Lauf sich etwas geändert hat.
Die Änderungen selber kann man dann auch leicht im git darstellen.

Die Dauer eines Abgleich ist naturgemäß schwierig allgemeingültig darzustellen. Ein Anhaltswert: auf einem Raspberry 3B+ auf SD-Karte mit ca. 30 Scripten, 2000 Variablen und 20 Modulen dauert ohne Erstellen von Zip-Archiven ca. 20 Sekunden, jeder weiterer Abgleich ca. 2-3 Sekunden. Mit Erstellen von Zip-Archiven wächst die Zeit initial auf 150 Sekunden.

Da alle Änderungen von IPS bei Abgleich in das lokale Repository übertragen und direkt an das zentrale Repository übertragen werden, ist dieses Verzeichnis nur von temporärem Interesse und kann jderzeit neu erstellt werden.

## 2. Voraussetzungen

 - IP-Symcon ab Version 5

## 3. Installation

Die Konsole von IP-Symcon öffnen. Im Objektbaum unter Kerninstanzen die Instanz __*Modules*__ durch einen doppelten Mausklick öffnen.

In der _Modules_ Instanz rechts oben auf den Button __*Hinzufügen*__ drücken.

In dem sich öffnenden Fenster folgende URL hinzufügen:

`https://github.com/demel42/IPSymconConfigVC.git`

und mit _OK_ bestätigen.

Anschließend erscheint ein Eintrag für das Modul in der Liste der Instanz _Modules_

### Einrichtung in IPS

In IP-Symcon nun _Instanz hinzufügen_ (_CTRL+1_) auswählen unter der Kategorie, unter der man die Instanz hinzufügen will, und Hersteller _(sonstiges)_ und als Gerät _Configuration Version-Control_ auswählen.

### Einrichtung des Git-Repositories

##### lokaler Git-Server auf raspbian und ubuntu

siehe auch `https://git-scm.com/book/de/v1/Git-auf-dem-Server-Einrichten-des-Servers` und `https://www.linux.com/learn/how-run-your-own-git-server`.

Der User _git_ ist Platzhalter, ebenso _git-server_ oder _ipsymcon.git_ bzw _ipsymcon_. Die Namen bzw Pfad können natürlich nach Bedarf angepasst werden.

Bei einem Raspberry sollte man vielleicht den Git-Server-Repository also auch das geclone Git-Repository nicht auf der internen SD-Karte anlegen.

**auf dem Git-ServerGit-Server**
```
sudo -i
adduser git
<passwort eingeben und merken>
mkdir -p ~git/repositories/ipsymcon.git
cd ~git/repositories/ipsymcon.git
git init --bare
chown -R git:users ~git/repositories
```

**auf dem IPS-Server**<br>
Hintergrund: damit es richtig funktioniert muss mindestens eine Datei im Git-Repository vorhanden sein.

```
cd /tmp
git clone ssh://git@git-server/home/git/repositories/ipsymcon.git
cd ipsymcon
touch README.md
git add README.md
git commit -m "initial revision"
git push
cd /tmp
/bin/rm -rf /tmp/ipsymcon
```

#### Git-Server auf Synology DiskStation

**auf der Synology DiskStation**

Paket 'Git Server' installieren

Login als User _admin_
```
cd /volume1/git
mkdir ipsymcon.git
cd ipsymcon.git
git init --bare
```

**auf dem IPS-Server**

Der Git-User kann aber muss nicht _admin_ sein. Der ssh-Port ist typischerweise nicht _22_, sondern z.B. _5002_.

```
cd /tmp
git clone ssh://admin@git-server:5002/home/git/repositories/ipsymcon.git
cd ipsymcon
touch README.md
git add README.md
git commit -m "initial revision"
git push
cd /tmp
/bin/rm -rf /tmp/ipsymcon
```

#### Git-Repository auf öffentlichen Git-Server (wie _GtHub_)

Wichtig: das Repository muss auf jeden Fall als **_privat_** eingerichtet werden! Ein lokaler Git-Server ist natürlich zu bevorzugen.

Bei der Anlage des Repository sollte die Option gewählt werden, direkt ein leeres _README.md_ anzulegen.


#### ssh-Keys einrichten für lokale ssh-basierte Repositories

**auf dem IPS-Server**

Sowohl als User _pi_ als auch als User _root_ (IPS läuft ja als _root).

- falls keine Datei _~/.ssh/authorized_keys_ vorhanden ist
```
ssh-keygen -t rsa -b 2048
```
Die Fragen alle mit <return> beantworten.

Verteilen des ssh-Key auf den Git-Server:
```
ssh-copy-id git@git-server
```

Als User den oben gewählten Git-User verwenden.



#### Einrichtung auf dem IPS-Server
```
mkdir <Verzeichnis für lokales respoitory>
```


## 4. Funktionsreferenz

### zentrale Funktion

`boolean CVC_CloneRepository(integer $InstanzID)`<br>
legt ein frischen Clone des Repositories an. Dabei wird, wenn erforderlich, ein vorhænderen Clone gelöscht. Das angegebene lokale Verzeichnis muss vorhanden sein.<br>

`boolean CVC_PerformAdjustment(integer $InstanzID, boolean $with_zip)`<br>
führt einen Abgleich durch. Durch _with_zip_ kann bei manuellem Aufruf gesteuert werden, ob überhaupt Zip-Archive gebildet werden oder nicht (wirkt nur, _Module als Zip_ gesetzt ist).

## 5. Konfiguration:

### Variablen

| Eigenschaft                     | Typ      | Standardwert | Beschreibung |
| :-----------------------------: | :-----:  | :----------: | :----------------------------------------------------------------------------------------------------------: |
| Git-Repository                  | string   |              | Pfad zu Git-Repository |
| Benutzer                        | string   |              | Benutzer des Git-Repository (für https und ssh) |
| Passwort                        | string   |              | Passwort (nur für https) |
| Port                            | integer  | 22           | SSH-Port (nur für ssh) |
| lokales Verzeichnis             | string   |              | lokales Verzeichnis indem der Clone des Git-Repository abgelegt wird |
| Module als Zip-Archiv sichern   | boolean  | false        | Sichern der Module als Zip-Archiv. Achtung: Größe beachten! |

#### Schaltflächen

| Bezeichnung                  | Beschreibung |
| :--------------------------: | :-------------------------------------------------: |
| Abgleich durchführen         | führt einen Abgleich durch |
| Repository einrichten        | erzeugt einen aktuelle Clone des Repositories |

## 6. Anhang

GUIDs

- Modul: `{EA906449-5DDA-4D99-9110-276BB31B6FFC}`
- Instanzen:
  - ConfigVC: `{396EA137-2E5F-413A-A996-D662158EA481}`

## 7. Versions-Historie

- 1.0 @ 01.09.2018<br>
  Initiale Version
