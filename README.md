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

Dieses Repo dient erst einmal der Unterstützung des derzeitigen
[Berliner Amateurfunk-Kurses](https://www.chaoswelle.de/Kurs) (Stand:
Oktober 2016).

Ich (DJ3EI) biete an, kursbegleitend entsprechende Dateien zur
Verfügung zu stellen. Ich habe aus mehreren früheren Kursen Skripte,
die `.qlt`-Generierung relativ bequem machen.

Ich habe weiter Listen zusammengehöriger Fragen. (Denn was sinnvoll
zusammen gelernt wird, steht im Fragenkatalog leider nicht unbedingt
zusammen.)  Allerdings waren meine Kurse vom Stoffverlauf chaotischer
"organisiert". Eckarts Gliederung war mir schnurts, sondern ich habe
zum Beispiel gerne mit dem (appetitanregenden) Thema Wellenausbreitung
angefangen. Also können wir meine (durchaus vorhandenen) `.qlt` -
Dateien aus dem letzten Kurs nicht einfach nehmen.

## QLT-Dateien

*Die hier gelieferten QLT-Dateien sind noch nicht komplett.* Was es
schon gibt, liegt im `data` Verzeichnis.  Die Software, um diese
Dateien zu erzeugen, steckt komplett in zwei `Rakefile`.

Alles in diesem Repo steht unter der Apache-Lizens, siehe LICENSE.txt.

## Initiale Ernte

In der Datei [harvest.json](harvest.json) finden sich alle Fragen, die
im Onlinekurs direkt vorkommen.

Die Software, diese Liste von der Webseite des DARC zu extrahieren,
steckt in `harvest/Rakefile`. Man braucht ein halbwegs aktuelles Ruby
und das Gem `Nokogiri` installiert, wenn man das laufen lassen will;
Aufruf schlicht `rake`.

## Generierung der QLT-Dateien

Im Hauptverzeichnis `rake` aufrufen erzeugt die QLT-Dateien im Verzeichnis
`data`.

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