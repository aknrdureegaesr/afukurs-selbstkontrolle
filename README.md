This is intended to help people studying for the German radio amateur
exam check what progress they are making.  So this is all in German.

## Wäre doch schade...

... wenn eine zur BNetzA zur Amateurfunkprüfung geht und alle Fragen
richtig beantwortet. Es sei denn, sie wollte das so.

Wenn es ihr dagegen nur darum ging, zu bestehen, hat sie zuviel Zeit
in die Vorbereitung gesteckt: Mehr, als nötig war. Schließlich darf
man bis zu knapp 1/4 der Fragen falsch beantworten und hat immer noch
bestanden.

Noch schlimmer wäre natürlich, wenn jemand zur Prüfung geht und
durchfällt. Den Aufwand fürs Nochmallernen und das Geld für die
Wiederholungsprüfung möchte man lieber woanders investieren.

Wie viel Aufwand soll man dann ins Lernen stecken? Nicht zu viel und
nicht zu wenig. Nur gerade so viel, dass man locker besteht. Aber
woher weiß man das?

Dabei hilft eine kontinuierliche
Lernfortschritts-Selbstkontrolle. Genau darum geht es hier.

## Prüfungssimulationen mit AfuP

Wir kennen die Fragen, die in der Prüfung kommen, aus den amtlichen
Fragenkatalogen, die man von der [Webseite der
BNetzA](http://www.bundesnetzagentur.de/cln_1432/DE/Sachgebiete/Telekommunikation/Unternehmen_Institutionen/Frequenzen/SpezielleAnwendungen/Amateurfunk/amateurfunk_node.html)
holen kann.

Junghard DF1IAF hat die in Prüfungssimulationssoftware
gegossen. Danke, Junghard!

Man kann _nach Kursende_ auf
[seiner Webseite](http://www.afup.a36.de/) Simulationen der Prüfung
online durchführen. Testen, ob man den Stoff gut genug kann, ehe man
zur Prüfung geht.

Aber das geht auch vorher schon, die ganze Zeit, kursbegleitend!

Dazu kann man sich bei Junghard ein
[offline-Programm](http://www.afup.a36.de/download/download.html)
herunterladen, AfuP.exe. Dieses Programm füttert man mit
"`.qlt`-Dateien". Das sind schlicht Listen der Fragen, die im Kurs
schon dran waren. Das Ergebnis: Simulationsprüfungen, die genau dem
Kursfortschritt entsprechen.  Genau, was man braucht, um zu wissen, wo
man steht.

## AfuP laufen lassen.

Leider ist `AfuP.exe` ein Windows-Programm. Glücklicherweise läuft es
auch unter Linux via WINE ganz manierlich (wenn man gängige Fonts
installiert hat).

Gibt es Mac-User im Kurs? Für die könnte man ein Linux in einem
virtuellen-Maschine-Image zur Verfügung stellen, in der Linux, WINE
und AfuP bereits installiert sind.

## Woher die .qlt-Dateien nehmen?

Dieses Repo diente ursprünglich der Unterstützung der 2016er Ausgabe
des [Berliner Amateurfunk-Kurses](https://www.chaoswelle.de/Kurs).

Es lässt sich aber auch für andere Kurse nutzen, die nach Eckarts
Büchern vorgehen.

## NEU: Foliensatz L

*Neu!* Seit Anfang 2020 gibt es einen neuen
Powerpoint-[Foliensatz](https://www.darc.de/der-club/distrikte/l/referat-fuer-aus-und-weiterbildung/)
des Distriktes L.

Ich persönlich (DJ3EI) bin mit diesem Foliensatz nicht recht glücklich:

* Die Folien sind pädagogisch und didaktisch nicht besonders geschickt
angelegt (das meinen ihre Autorinnen und Autoren durchaus auch
selbst - es sei halt ein erster Wurf).

* Die Wahl von Powerpoint als Tool macht die Zusammenarbeit unnötig
schwer.  Es gibt OpenSource-Folienmalwerkzeuge, die sich jedermensch
zumutbar besorgen kann, egal, ob für Linux, Mac oder Windows.
Powerpoint gehört nicht dazu.

* Die Folien sind einer OpenSource-Lizenz unterstellt. Das ist extrem
löblich und genau so sollte es sein!  Aber wer sie tatsächlich
weiterverwurstet, wie man es bei OpenSource-Material kann und darf,
riskiert Abmahnung, da an einzelnen Stellen "geklautes"
urheberrechtlich heikles Material benutzt wird (kommerzielle Comics).

Dazu kommt Kleinkram:

* Die unübersichtliche Nummerierung der Lektionen, zum Beispiel 
"1.2.2" → "1.2.3-1.2.5" → "1.2.6"

* Auf den ersten Blick sieht es so aus, als wären 1. Techniklektionen,
2. Betrieb und 3. Vorschriften, aber das wurde (glücklicherweise!) nicht
durchgehalten. Damit bräuchte man eine Reihenfolge der Lektionen.

* Die ZIP-Archive enthalten Dateinamen, die kodiert sind mit Mitteln
der Bronzezeit der Datenverarbeitung, damals, vor Unicode.

Aber die Folien haben immerhin den Vorteil, real zu existieren und es
gibt Kurse, die sie benutzen.  Daher werden sie hier unterstützt.

Das Reihenfolgenproblem löst der [Berliner (D23)
Plan](https://www.chaoswelle.de/Lehrgang_Berlin_2020/Unterrichtsplan_E).
Ich habe ein den Verdacht, dass die Stofflast auf den einzelnen
Abenden nicht gut verteilt ist, aber sei's drum.

## QLT-Dateien

finden sich in den Verzeichnissen [data](data), [berlin](berlin) und
[l-folien](l-folien).

Die Software, um diese Dateien zu erzeugen, ist in verschiedenen
Dateien mit Namen `Rakefile` kodiert.

Alles in diesem Repo steht unter der Apache-Lizens, siehe LICENSE.txt.

## Initiale Ernte Moltrecht

In der Datei [harvest.json](harvest.json) finden sich alle Fragen, die
im Onlinekurs direkt vorkommen.

Die Software, diese Liste von der Webseite des DARC zu extrahieren,
steckt in `harvest/Rakefile`. Man braucht ein halbwegs aktuelles Ruby
und das Gem `Nokogiri` installiert, wenn man das laufen lassen will;
Aufruf dann schlicht `rake`.

## Ernte L-Material

In der Datei [l-material.yaml](l-material.yaml) findet sich die Liste
der Lektionen aus dem L-Material sowie die Information, welche 
Powerpoint-Lektion welche Fragen enthält.  Um diese Datei neu zu
generieren, ruft man `rake harvest_l` auf; es wird dann die
entsprechende
[Tabelle](https://www.darc.de/der-club/distrikte/l/referat-fuer-aus-und-weiterbildung/)
heruntergeladen und analysiert.

## Generierung der QLT-Dateien

Im Hauptverzeichnis `rake` aufrufen erzeugt die QLT-Dateien in den Verzeichnissen
`data`, `berlin` und `l-folien`.

## Dateien

### `alle_*.qlt`

Diese Dateien dienen der Kontrolle, ob keine Fragen vergessen worden
sind.  Sie wurden mit dem Prüfungseditor von `AfuP` einmalig erzeugt
und hier eingecheckt.

### `conflict_resolution.yaml`

Die Logik ist sich nicht immer sicher, zu welcher Kurslektion eine
Frage gehört.  Hier kann das geklärt werden, indem diese Datei
händisch editiert wird.  Die Datei enthält Hintergrundinformationen,
die vom Programm ignoriert werden, aber für Menschen vielleicht ganz
nützlich sind.

### `harvest.json` und `te_harvest.json`

Aus Herunterladen der Webseite stark eingedampfte Information, welche
Frage in welcher Lektion gefunden wurde.  Diese Datei wird komplette
automatisch erzeugt, sie bitte nicht editieren.

### `learn_batches.yaml`

Material aus früheren Kursen, die einer ganz anderen Gliederung
gefolgt sind.  Hier sind Fragen zu zusammengehörigen Fragenblöcken
gegliedert, die sich gut zusammen unterrichten lassen.

Die Datei ist aus dem Datenbestand des früheren Kurses automatisch
erzeugt worden.

### `otherwise_forgotten.yaml`

Eine sehr schlichte Datei mit Frage-Lektions-Zuordnung.

Ob man eine bestimmte Frage hier oder in `conflict_resolution.yaml`
einer Lektion zuweist, ist im Ergebnis gleichgültig.