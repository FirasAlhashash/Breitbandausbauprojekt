# Breitbandausbauprojekt

Studienprojekt im Team. Mit Daten aus dem Breitbandatlas wollten wir herausfinden, welche Landkreise in Deutschland beim weiteren Breitbandausbau Vorrang haben könnten.

## Aufgabe

Die Aufgabe war als Rollenspiel aufgebaut: Es gab eine Ausschreibung ([`Fiktive_Ausschreibung.pdf`](Fiktive_Ausschreibung.pdf)), auf die wir ein Angebot ([`Fiktives_Angebot.pdf`](Fiktives_Angebot.pdf)) geschrieben haben. Beide Unterlagen sind fiktiv. Die Bundesnetzagentur und die Firma im Angebot haben nichts damit zu tun. Unser Ergebnis haben wir in [`finale presentation.pdf`](finale%20presentation.pdf) vorgestellt.

## Team

- Firas Alhashash: Projektmanagement – Organisation, Scrum-Management und Teamunterstützung
- Kyrill Kolosov: Entwicklung – Programmierung und Implementierung
- Miral Ibrahim: Datenanalyse
- Ibrokhim Eshanov: Qualitätssicherung und Visualisierungen
- Zayd Maatouf: Dokumentation

## Mein Beitrag

Ich habe das Projekt organisiert, das Scrum-Management übernommen und das Team unterstützt.

Der Projektcode steht in [`PLMP_firas finall.ipynb`](PLMP_firas%20finall.ipynb).

## Daten

| Datei | Inhalt | Quelle | Stand |
|---|---|---|---|
| [`data/bba_01_2024.xlsx`](data/bba_01_2024.xlsx), [`data/Schulen.csv`](data/Schulen.csv), [`data/Privathaushalte.csv`](data/Privathaushalte.csv) usw. | Breitband- und Mobilfunkverfügbarkeit nach Landkreisen | Breitbandatlas (bestätigt) | Festnetz 12.2023, Mobilfunk 01.2024 |
| [`data2für trainieren/`](data2f%C3%BCr%20trainieren) | dieselben Tabellen in einem neueren Stand | Breitbandatlas (vermutet) | Festnetz 06.2024 |
| [`data/Miral_neu.csv`](data/Miral_neu.csv) | Anzahl Förderverfahren pro Landkreis (`wieOft_Landkreis`) | Förderlandkarte (vermutet) | unbekannt |
| [`data/firas2.csv`](data/firas2.csv) | Niederlassungen pro Landkreis | nicht dokumentiert | unbekannt |

- [Breitbandatlas der Bundesnetzagentur](https://www.bundesnetzagentur.de/GIGA/DE/Breitbandatlas/start.html), die Statistik-Dateien `bba_MM_JJJJ.xlsx` gibt es im [Gigabit-Grundbuch](https://gigabitgrundbuch.bund.de/GIGA/DE/Downloads_Suche/start.html)
- [Förderlandkarte des BMDV](https://www.govdata.de/suche/daten/forderlandkarte-des-bmdv)

`wieOft_Landkreis` ist die Zielgröße: wie oft ein Landkreis in einer Liste von Breitband-Förderverfahren vorkommt. Der Export, aus dem gezählt wurde, liegt nicht im Repo. 320 der 400 Landkreise haben einen Eintrag.

## Vorgehen

1. Tabellen aus dem Breitbandatlas einlesen, bereinigen und auf Landkreise filtern
2. KMeans-Clustering (k = 3) der Landkreise nach Mobilfunk- und Breitbandabdeckung
3. Tabellen über den Gemeindeschlüssel (AGS) zusammenführen
4. Random Forest (`max_depth=5`) sagt `wieOft_Landkreis` aus dem Anteil mit ≥ 1000 Mbit/s bei Schulen, Unternehmen, Gewerbegebieten und Privathaushalten vorher
5. Modell auf den neueren Datenstand anwenden, Ergebnis in [`visual_df.csv`](visual_df.csv)

## Notebook starten

```bash
python -m venv .venv
.venv\Scripts\activate          # Linux/macOS: source .venv/bin/activate
pip install -r requirements.txt
pip install notebook
jupyter notebook "PLMP_firas finall.ipynb"
```

Das Notebook aus dem Repo-Ordner starten, weil die Dateien über relative Pfade geladen werden. Es braucht pandas 2.x (siehe [`requirements.txt`](requirements.txt)).

## Ergebnisse

Bewertung auf 20 % der Landkreise als Testdaten (`random_state=0`):

| | MAE | RMSE | R² |
|---|---:|---:|---:|
| Mittelwert aus dem Training | 7,88 | 10,71 | −0,09 |
| Random Forest | 6,71 | 10,32 | −0,01 |

## Grenzen

- Das Modell ist auf den Testdaten kaum besser als der Mittelwert.
- Die Herkunft der Förderzählungen ist nicht bestätigt.
- Die 80 fehlenden Landkreise wurden mit 0 Förderverfahren gerechnet. Das ist eine ungesicherte Annahme.
- Die Ergebnisse belegen keinen zukünftigen Förderbedarf.

## Überarbeitung (November 2026)

Das Repo wurde aufgeräumt: doppelte Dateien entfernt, Daten nach [`data/`](data) verschoben, Tabellen über den AGS statt über die Zeilenposition verbunden und fehlende Werte nicht mehr pauschal mit 0 gefüllt. Die fehlende `Aktuelle_daten.csv` wird jetzt aus [`data2für trainieren`](data2f%C3%BCr%20trainieren) aufgebaut, und das Modell wird mit einer Train/Test-Aufteilung bewertet. Die Notebooks in [`data_mashine_learning_project/`](data_mashine_learning_project) sind ältere Entwürfe und wurden nicht angepasst.
