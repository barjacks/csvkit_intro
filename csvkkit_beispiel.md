# Einführung in CSVkit, ein Zusatzprogramm der Kommandozeile

## Erste Schritte
- Machen wir zuerst einen Ordner, um unsere Daten abzulegen. ```mkdir data```
- Lade nun von [diesem Link](https://www.dropbox.com/s/ml38emb5i41w9bl/%C3%B6v_nodups.csv?dl=0) diese Daten in den eben geschaffenden Ordner herunter.
- Navigiere in diesen Order ```cd data```.
- Schauen wir an, was wir hier haben ```csvcut öv_nodups.csv```
- Nicht sehr ansehnlich. Wir können nun nur die Kopfzeile anschauen. Das machen wir mit ```csvcut -n öv_nodups.csv```.
- Speichern wir diese Kopfzeile einmal ab: ```csvcut -n öv_nodups.csv > kopfzeile.txt```
- Öffnen wir das File nun mit ```atom kopfzeile.txt```.

## Lassen wir die Daten zusammenfassen
- Schauen wir uns die Zusammenfassungen aller Daten einmal an. ```csvstat öv_nodups.csv```
- Weil das eine Weile dauert, macht es Sinn, das abzuspeichern. ```csvstat öv_nodups.csv > anaylse_öv.txt```
- Öffnen wir die Zusammenfassung nun wieder mit ```atom anaylse_öv.txt```.
- Damit können wir unsere Fragestellungen formulieren. Vielleicht folgendes:

1. Nehmen die Ereignisse eigentlich zu oder ab? "Jahre"
2. Wie entwickeln sich die Erreignisse nach:
  - "E_Verkehrsunternehmung1": "SBBC" und "SBBP"
  - "E_Verkehrsart": "Strassenbahn", "Trolleybus"
  - "Klassifizierung_WAS": Suizid
3. An welchem Tag passierten die meisten Ereignisse?
4. ... ?

## 1. Nehmen die Ereignisse eigentlich zu oder ab? "Jahre"

- ```csvstat -c "Jahre" --freq-count 10 öv_nodups.csv```
- Um es abzuspeichen ```csvstat -c "Jahre" --freq-count 10 öv_nodups.csv > 1.txt```
- In Google Sheets weiter damit arbeiten.

## 2. Wie entwickeln sich die Ereignisse nach "E_Verkehrsunternehmung1" / "E_Verkehrsart" / "Klassifizierung_WAS"?

- "E_Verkehrsunternehmung1": ```csvstat -c "E_Verkehrsunternehmung1" --freq-count 10 öv.csv```
- Jetzt wählen wir nur die Kolonnen mit **"SBBC"** und verbinden es mit einer csvstat abfrage. Das wir mit einer sogenannten Pipe. "|" ```csvgrep -m 'SBBC' -c 'E_Verkehrsunternehmung1' öv_nodups.csv | csvstat -c "Jahre" --freq-count 10```
und: ```csvgrep -m 'SBBP' -c 'E_Verkehrsunternehmung1' öv_nodups.csv | csvstat -c "Jahre" --freq-count 10```
- Wie steht es mit den Verkehrsarten: **Strassenbahn** und **Trolleybus**? ```csvgrep -m 'Strassenbahn' -c 'E_Verkehrsart' öv_nodups.csv | csvstat -c "Jahre" --freq-count 10``` und ```csvgrep -m 'Trolleybus' -c 'E_Verkehrsart' öv_nodups.csv | csvstat -c "Jahre" --freq-count 10```
- Und schliesslich **Suizid**: ```csvgrep -m 'Suizid' -c 'Klassifizierung_WAS' öv_nodups.csv | csvstat -c "Jahre" --freq-count 10```
- Um die Ergebnisse auszuplotten, am besten speichern und dann bei Google Spreadsheets bearbeiten.

## 3. An welchem Tag passierten die meisten Ereignisse?

- Zählen wir, an welche Datum die meisten Ereignisse passierten```csvstat -c "E_Datum" --freq-count 10 öv_nodups.csv```
- Das genügt uns nicht. Denn hier ist auch die Zeit dabei. Diese Zeit muss weg.
- Zuerst filtern wir nur die Daten heraus: ```csvcut -c "E_Datum" öv_nodups```
- Nun müssen wir die Daten beschneiden. Das tun wir mit dieser kryptischen Zeile, die wir von [hier zusammengeklauen](https://www.ibm.com/developerworks/aix/library/au-unixtext/index.html). ```awk -F: '{printf("%s \n", substr($1,1,10))}'```
- Um zu testen arbeiteten wir am Ende noch mit ```|head -5```
- Also alles zusammen: ```csvcut -c "E_Datum" öv_nodups.csv |awk -F: '{printf("%s \n", substr($1,1,10))}'|head -5```
- Und jetzt können wir einfach zählen, wie wir das bereits getan haben. Jetzt ohne head. Die ganze Zeile sieht also so aus: ```csvcut -c "E_Datum" öv_nodups.csv |awk -F: '{printf("%s \n", substr($1,1,10))}'|csvstat -c "E_Datum" --freq-count 10```
- Schauen wir uns diesen Tag an: ```csvgrep -r '^2012-01-20' -c 'E_Datum' öv_nodups.csv```

## 4. ... ?
