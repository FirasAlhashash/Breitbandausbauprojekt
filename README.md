# Breitbandausbauprojekt

Studienprojekt im Team. Wir sollten mit Daten aus dem Breitbandatlas herausfinden, welche Landkreise in Deutschland beim nächsten Breitbandausbau Vorrang haben könnten.

## Aufgabe

Das Projekt war als Rollenspiel aufgebaut. Es gab eine Ausschreibung (`Fiktive_Ausschreibung.pdf`) und ein Angebot (`Fiktives_Angebot.pdf`). **Beide Unterlagen sind fiktiv.** Die Bundesnetzagentur und die Firma im Angebot haben damit nichts zu tun, das steht auch in den PDFs selbst. Auch die Firmenangaben auf der letzten Folie der Präsentation gehören zum Rollenspiel.

Die Arbeitspakete laut Ausschreibung waren Datenexploration, ein Konzept, die Datenanalyse und am Ende eine Empfehlung für eine Förderstrategie. Das Ergebnis haben wir in `finale presentation.pdf` vorgestellt.

**Team** (laut Abschlusspräsentation): Firas Alhashash (Entwicklung), Kyrill Kolosov (Projektmanagement), Miral Ibrahim (Datenanalyse), Ibrokhim Eshanov (Qualitätssicherung und Visualisierungen), Zayd Maatouf (Dokumentation).

**Mein Teil:** Laut Präsentation war ich für die Umsetzung der Programmier- und Implementierungsaufgaben zuständig. Der Code dazu steht in `PLMP_firas finall.ipynb`.

## Daten

| Datei | Inhalt | Stand |
|---|---|---|
| `data/*.csv` | Breitband- und Mobilfunkverfügbarkeit pro Gebiet (Fläche, Privathaushalte, Schulen, Unternehmen, Gewerbegebiete, ...) | Festnetz 12.2023, Mobilfunk 01.2024 (laut Dateiköpfen) |
| `data/bba_01_2024.xlsx` | Original-Download aus dem Breitbandatlas | 01.2024 laut Dateiname |
| `data2für trainieren/*.csv` | dieselben Tabellen in einem neueren Stand | Festnetz 06.2024 (laut Kopf von Gewerbegebiete und Privathaushalte) |
| `data/Miral_neu.csv` | `wieOft_Landkreis` pro Landkreis (siehe unten) | nicht dokumentiert |
| `data/firas2.csv` | Anzahl Niederlassungen pro Landkreis, aus `regionaleWirtschaftsstruktur.csv` | nicht dokumentiert |

Als Quellen nennt die Präsentation das [Gigabit-Grundbuch](https://gigabitgrundbuch.bund.de/GIGA/DE/Downloads_Suche/start.html) (Breitbandatlas-Downloads) und die [Förderlandkarte des BMDV](https://bmdv.bund.de/DE/Themen/Foerderlandkarte/foerderlandkarte.html). Für `Miral_neu.csv` und `regionaleWirtschaftsstruktur.csv` steht im Repo nicht genau, woher die Zahlen kommen.

### Was bedeutet `wieOft_Landkreis`?

`wieOft_Landkreis` gibt an, wie oft ein Landkreis in einer Liste von Breitband-Förderverfahren vorkommt, also wie viele Förderverfahren es dort gab. Das ist die Zielgröße des Modells.

- `Miral_neu.csv` enthält 320 der 400 Landkreise, mit Werten von 1 bis 149.
- Die übrigen 80 Landkreise stehen nicht in der Liste und bekommen den Wert 0, also kein Förderverfahren. Das ist eine Annahme: Sie stimmt nur, wenn die Liste vollständig ist.
- Die Zahlen in `Miral_neu.csv` unterscheiden sich von der Datei `Förderungsverfahren_geplant_2024.xls` im Ordner `data_mashine_learning_project/`. Welche Liste genau dahintersteht, ist nicht dokumentiert.

## Vorgehen

Alles steht in `PLMP_firas finall.ipynb`:

1. Tabellen aus dem Breitbandatlas einlesen, Kopfzeilen bereinigen, auf Landkreise filtern (`3 - Kreis`), Dezimalkommas umwandeln
2. KMeans-Clustering (k = 3, nach Ellbogen-Methode) der Landkreise nach Mobilfunkabdeckung in der Fläche und nach Breitband bei Privathaushalten
3. Daten über den Gemeindeschlüssel (AGS) zusammenführen
4. Random Forest (`max_depth=5`) sagt `wieOft_Landkreis` aus dem Anteil mit ≥ 1000 Mbit/s bei Schulen, Unternehmen, Gewerbegebieten und Privathaushalten vorher
5. Das Modell auf den neueren Datenstand anwenden und das Ergebnis als `visual_df.csv` speichern. Die interaktive Karte aus der Präsentation (folium) ist nicht im Repo.

## Notebook starten

```bash
python -m venv .venv
.venv\Scripts\activate          # Linux/macOS: source .venv/bin/activate
pip install -r requirements.txt
pip install notebook
jupyter notebook "PLMP_firas finall.ipynb"
```

Das Notebook muss aus dem Repo-Ordner gestartet werden, weil die Dateien über relative Pfade (`data/...`) geladen werden. Es braucht pandas 2.x: Die Funktion `clean_data` greift mit `df.iloc[0][i]` über die Position zu, und das funktioniert in pandas 3 nicht mehr.

Getestet mit Python 3.11 und den Versionen aus `requirements.txt`: Kernel neu gestartet, alle Zellen von oben nach unten ausgeführt, ohne Fehler.

## Ergebnisse

Die Daten werden 80/20 in Training und Test aufgeteilt (`random_state=0`, 320 bzw. 80 Landkreise). Bewertet wird nur auf dem Testteil:

| | MAE | RMSE | R² |
|---|---:|---:|---:|
| Mittelwert aus dem Training (immer 9,18) | 7,88 | 10,71 | −0,09 |
| Random Forest | 6,71 | 10,32 | −0,01 |

Der Random Forest ist nur wenig besser als eine Vorhersage, die immer den Durchschnitt nimmt. Ein R² um 0 heißt, dass das Modell die Unterschiede zwischen den Landkreisen praktisch nicht erklärt.

Ich habe das auch mit anderen Aufteilungen geprüft (Nebenrechnung, nicht im Notebook). Mit fünf verschiedenen `random_state`-Werten liegt R² zwischen −0,10 und +0,10. Bei einer 5-fachen Kreuzvalidierung ist R² im Mittel 0,03.

In der ersten Version wurde das Modell auf denselben Daten bewertet, auf denen es trainiert wurde (MSE 101 gegenüber 217 beim Mittelwert). Auf den Trainingsdaten erreicht es R² = 0,53, auf neuen Daten aber nicht. Der Unterschied kam also vom Auswendiglernen.

Nach der Feature Importance des Random Forest ist der Anteil der Privathaushalte mit ≥ 1000 Mbit/s am wichtigsten (0,42), danach kommen Gewerbegebiete (0,26), Schulen (0,20) und Unternehmen (0,12). Weil das Modell kaum etwas vorhersagt, sollte man diese Reihenfolge aber nicht überbewerten.

## Grenzen

- **Die Vorhersage in `visual_df.csv` ist kein nachgewiesener zukünftiger Förderbedarf.** Das Modell schätzt, wie viele Förderverfahren ein Landkreis mit dieser Breitbandversorgung bisher ungefähr gehabt hätte. Auf den Testdaten ist es kaum besser als der Durchschnitt.
- Die Zielgröße zählt Förderverfahren. Wie groß oder teuer ein Verfahren ist, spielt dabei keine Rolle.
- Für 80 Landkreise ist `wieOft_Landkreis = 0` eine Annahme (siehe oben).
- Mit 400 Landkreisen ist der Datensatz klein, und genutzt werden nur vier Merkmale.
- Bevölkerung, Kosten oder geplante eigenwirtschaftliche Ausbauten sind nicht berücksichtigt.

## Was überarbeitet wurde (November 2026)

- **Doppelte Dateien entfernt:** Ich habe nur Dateien gelöscht, die Byte für Byte identisch waren. Das waren elf Kopien von `bba_01_2024.xlsx`, zehn `.xls`-Kopien der CSV-Dateien und zwei Kopien von `firas2`. Die benötigten Dateien liegen jetzt in `data/`, die Pfade im Notebook sind angepasst.
- **`Aktuelle_daten.csv` fehlte im Repo.** Das Notebook baut die Tabelle jetzt aus `data2für trainieren` neu auf. Zur Kontrolle: Mit einem Modell auf allen Daten, wie in der ersten Version, kommen genau die Werte aus der alten `visual_df.csv` heraus.
- **Zuordnung über AGS:** Einige Spalten wurden bisher nur über die Zeilenposition zugeordnet. Bei einer Ausführung von oben nach unten passte das zufällig, weil alle Tabellen gleich sortiert sind. Jetzt läuft die Zuordnung über den AGS, mit einer Prüfung auf doppelte Schlüssel.
- **Fehlende Werte:** Sie werden nicht mehr pauschal mit 0 gefüllt, nur noch `wieOft_Landkreis` (siehe oben).
- **Bewertung:** Das Modell wird jetzt mit einer Train/Test-Aufteilung bewertet, mit MAE, RMSE und R² und dem Mittelwert als Vergleich.
- **Ältere Entwürfe:** Die Notebooks in `data_mashine_learning_project/` sind ältere Entwürfe und wurden nicht angepasst. Ihre Dateipfade stimmen nicht mehr.
