# LinkedIn Job-Scraper

Diese Anleitung erklärt, wie Sie dieses Repository verwenden können, um mit der Scrapfly-API Jobdaten von LinkedIn zu scrapen.

## Einrichtung

Dieser Linkedin.com-Scraper verwendet **Python 3.10** mit dem [scrapfly-sdk](https://pypi.org/project/scrapfly-sdk/)-Paket, das zum Scrapen und Parsen von LinkedIn-Daten verwendet wird.

1.  Stellen Sie sicher, dass Sie **Python 3.10** und den [Poetry](https://python-poetry.org/docs/#installation) Python-Paketmanager auf Ihrem System installiert haben.
2.  Holen Sie sich Ihren Scrapfly-API-Schlüssel von [https://scrapfly.io/dashboard](https://scrapfly.io/dashboard) und setzen Sie die `SCRAPFLY_KEY`-Umgebungsvariable:
    > **Hinweis:** Scrapfly bietet einen kostenlosen Tarif mit 1.000 API-Credits für den Einstieg. Bezahlte Pläne sind ebenfalls verfügbar, falls Sie mehr Credits benötigen.

    ```shell
    export SCRAPFLY_KEY="DEIN SCRAPFLY SCHLÜSSEL"
    ```
3.  Klonen Sie dieses Repository und installieren Sie die Python-Umgebung:
    ```shell
    git clone https://github.com/scrapfly/scrapfly-scrapers.git
    cd scrapfly-scrapers/linkedin-scraper
    poetry install
    ```

## Scrapen von Jobs

Das `linkedin.py`-Skript bietet zwei Hauptfunktionen zum Scrapen von Jobdaten:

1.  `scrape_job_search()`: Um Jobs anhand von Schlüsselwörtern und Standorten zu finden. Diese Funktion gibt eine Liste von Jobs mit zusammenfassenden Daten wie Titel, Firma, Standort und der Job-URL zurück.
2.  `scrape_jobs()`: Um detaillierte Informationen zu bestimmten Jobs anhand ihrer URLs zu scrapen.

### Jobsuche

Um nach Jobs zu suchen, können Sie die Funktion `scrape_job_search()` verwenden, die ein `keyword` (Schlüsselwort), einen `location` (Standort) und einen optionalen `max_pages`-Parameter entgegennimmt, um die Anzahl der zu scrapenden Suchergebnisseiten zu begrenzen.

Hier ist ein Anwendungsbeispiel:

```python
import asyncio
from linkedin import scrape_job_search

async def main():
    job_ergebnisse = await scrape_job_search(
        keyword="Python Entwickler",
        location="Deutschland",
        max_pages=1  # Nur die erste Ergebnisseite scrapen
    )
    print(job_ergebnisse)

if __name__ == "__main__":
    asyncio.run(main())
```

### Einzelne Jobseiten

Sobald Sie eine Liste von Job-URLs aus der Jobsuche haben, können Sie die Funktion `scrape_jobs()` verwenden, um detaillierte Informationen für jeden Job abzurufen. Diese Funktion benötigt eine Liste von Job-URLs.

Beispiel:

```python
import asyncio
from linkedin import scrape_jobs

async def main():
    job_details = await scrape_jobs(
        urls=[
            "https://www.linkedin.com/jobs/view/python-developer-at-tactibit-technologies-4121519145",
            "https://www.linkedin.com/jobs/view/python-developer-remote-position-at-hrc-global-services-4300086838",
        ]
    )
    print(job_details)

if __name__ == "__main__":
    asyncio.run(main())
```

## Alles zusammengefügt: Vollständiges Beispiel

Hier ist ein komplettes Skript, das nach Jobs sucht, die detaillierten Informationen für jeden gefundenen Job abruft und die Ergebnisse in JSON-Dateien speichert.

Erstellen Sie eine neue Datei, zum Beispiel `run_job_scraper.py`, und fügen Sie den folgenden Code ein:

```python
import asyncio
import json
from pathlib import Path
import linkedin

# Ein Verzeichnis erstellen, um die Ergebnisse zu speichern
output = Path(__file__).parent / "job_ergebnisse"
output.mkdir(exist_ok=True)

async def main():
    print("Starte LinkedIn-Jobsuche...")
    # Zuerst Jobs über die Suche finden
    job_suchergebnisse = await linkedin.scrape_job_search(
        keyword="Data Scientist",
        location="Deutschland",
        max_pages=1  # Für dieses Beispiel nur die erste Seite scrapen
    )

    # Die Suchergebnisse speichern
    search_output_path = output / "job_suche.json"
    with open(search_output_path, "w", encoding="utf-8") as f:
        json.dump(job_suchergebnisse, f, indent=2, ensure_ascii=False)
    print(f"{len(job_suchergebnisse)} Job-Suchergebnisse in {search_output_path} gespeichert")

    # Nun die detaillierten Informationen für jeden gefundenen Job scrapen
    job_urls = [job["jobUrl"] for job in job_suchergebnisse]

    if not job_urls:
        print("Keine Job-URLs zum Scrapen gefunden.")
        return

    print(f"Scrape Details für {len(job_urls)} Jobs...")
    job_details = await linkedin.scrape_jobs(urls=job_urls)

    # Die detaillierten Jobdaten speichern
    details_output_path = output / "job_details.json"
    with open(details_output_path, "w", encoding="utf-8") as f:
        json.dump(job_details, f, indent=2, ensure_ascii=False)
    print(f"Detaillierte Informationen für {len(job_details)} Jobs in {details_output_path} gespeichert")


if __name__ == "__main__":
    # Stellen Sie sicher, dass Sie Ihre SCRAPFLY_KEY-Umgebungsvariable setzen, bevor Sie das Skript ausführen
    asyncio.run(main())
```

Um dieses Skript auszuführen, speichern Sie es und führen Sie es dann von Ihrem Terminal aus (nachdem Sie die Umgebung wie oben beschrieben eingerichtet haben):

```shell
poetry run python run_job_scraper.py
```