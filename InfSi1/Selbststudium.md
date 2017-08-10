Dieses Selbststudium bezieht sich auf zwei Heise-Artikel.

# Makroviren

Quelle: https://www.heise.de/security/artikel/Analysiert-Das-Comeback-der-Makro-Malware-2573181.html	

Dokument in E-Mail, man solle die Rechnung prüfen

Dokumente wie Word sind nicht standardmässig verdächtig, wie .exe oder .zip usw.

Wenn man das Dokument aufmacht, steht gross "Please enable macroses (!) to view the document"

Das Makro ist in VBA geschrieben, bei Open-/LibreOffice ging das Makro nicht.

Mit `Auto_Open()` bzw. `Workbook_Open()` wird eine Funktion automatisch ausgeführt, hier wird `xpress()` aufgerufen.

Es wird das TEMP-Verzeichnis (`%TEMP%`) ermittelt und ein Dateipfad konstruiert. Dabei wird aber der Slash vergessen und es ist somit nur ein langer Dateiname, der dann in `Appdata\Local` wandert.

Im Dokument selber ist dann ein String, beginnend mit `5678`, mit weisser Schrift auf weissem Hintergrund versteckt, bestehend aus Hex-Zeichen.

Dieser Text wird dann in Blöcken von je 4 Bytes in die Ausgabedatei geschrieben, und dann die Datei mit einem `Shell` Befehl gestartet.

Wenn man die entstandene Datei analysieren lässt, ist es eine Backdoor-Malware die den PC zum Andromeda- bzw. Gamarue-Botnet hinzufügt. 

Der Clou in dieser Malware besteht also vorallem in dem guten Tarnen der Malware-Payload als Text im HEX-Format.

# PDF-Malware

Quelle: https://www.heise.de/security/artikel/Analysiert-Alte-Masche-neue-Verpackung-Infektion-durch-PDFs-3722708.html

PDF per Spam-Mail, typischerweise "Receipt", "Payment" oder "Invoice"

Im PDF steckt eingebettetes JavaScript. Das JavaScript ist ziemlich umfangreich, dient aber zum grössten Teil nur zur Tarnung, es entstammt einer Anwendung zur Verwaltung von Wetterdaten. Tatsächlich aufgerufen werden nur wenige Funktionen.

Wenn man die Funktionsaufrufe nachverfolgt, kommt schlusssendlich folgendes zum Vorschein:

```javascript
exportDataObject ({ cName: "LAMIKSJZ.docm", nLaunch: 2 })
```

`exportDataObject` ist in der API vom Acrobat (Reader) spezifiziert, funktioniert aber auch auf anderen Readern.

`cName` ist in diesem Fall der Name des zu extrahierenden Datenobjekts; mit `nLaunch: 2` deklariert man dass die extrahierte Word-Datei in einem temporären Pfad gespeichert wird, und nach einer Sicherheitswarnung ausgeführt wird. Schliesst man den Reader wird die temporäre Kopie automatisch gelöscht.

Im Original wird diese Funktion von `stdOutS`  ausgeführt, diese Funktion wird vom PDF-Reader mittels im Dokument hinterlegten Eigenschaften automatisch ausgeführt.

Im Word-Dokument werden dann Makroviren aktiv, nachdem man aufgefordert wird Makros zuzulassen und die geschützte Ansicht von Word deaktiviert. Am Ende wird Locky heruntergeladen und der PC infiziert. 