# LinkedIn Job Scraper

This guide explains how to use this repository to scrape job data from LinkedIn using the Scrapfly API.

## Setup

This Linkedin.com scraper uses **Python 3.10** with the [scrapfly-sdk](https://pypi.org/project/scrapfly-sdk/) package, which is used to scrape and parse LinkedIn's data.

1.  Ensure you have **Python 3.10** and the [Poetry](https://python-poetry.org/docs/#installation) Python package manager on your system.
2.  Retrieve your Scrapfly API key from [https://scrapfly.io/dashboard](https://scrapfly.io/dashboard) and set the `SCRAPFLY_KEY` environment variable:
    ```shell
    export SCRAPFLY_KEY="YOUR SCRAPFLY KEY"
    ```
3.  Clone this repository and install the Python environment:
    ```shell
    git clone https://github.com/scrapfly/scrapfly-scrapers.git
    cd scrapfly-scrapers/linkedin-scraper
    poetry install
    ```

## Scraping Jobs

The `linkedin.py` script provides two primary functions for scraping job data:

1.  `scrape_job_search()`: To discover jobs based on keywords and location. This function returns a list of jobs with summary data like title, company, location, and the job URL.
2.  `scrape_jobs()`: To scrape detailed information about specific jobs using their URLs.

### Job Search

To search for jobs, you can use the `scrape_job_search()` function, which takes a `keyword`, `location`, and an optional `max_pages` parameter to limit the number of search result pages to scrape.

Here is an example of how to use it:

```python
import asyncio
from linkedin import scrape_job_search

async def main():
    job_results = await scrape_job_search(
        keyword="Python Developer",
        location="United States",
        max_pages=1  # Scrape only the first page of results
    )
    print(job_results)

if __name__ == "__main__":
    asyncio.run(main())
```

### Individual Job Pages

Once you have a list of job URLs from the job search, you can use the `scrape_jobs()` function to retrieve detailed information for each job. This function takes a list of job URLs.

Example:

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

## Putting It All Together: Full Example

Here is a complete script that searches for jobs, retrieves the detailed information for each job found, and saves the results to JSON files.

Create a new file, for example `run_job_scraper.py`, and paste the following code:

```python
import asyncio
import json
from pathlib import Path
import linkedin

# Create a directory to store the results
output = Path(__file__).parent / "job_results"
output.mkdir(exist_ok=True)

async def main():
    print("Starting LinkedIn job search...")
    # First, find jobs using search
    job_search_results = await linkedin.scrape_job_search(
        keyword="Data Scientist",
        location="Canada",
        max_pages=1  # Scrape only the first page for this example
    )

    # Save the search results
    search_output_path = output / "job_search.json"
    with open(search_output_path, "w", encoding="utf-8") as f:
        json.dump(job_search_results, f, indent=2, ensure_ascii=False)
    print(f"Saved {len(job_search_results)} job search results to {search_output_path}")

    # Now, scrape the detailed information for each job found
    job_urls = [job["jobUrl"] for job in job_search_results]

    if not job_urls:
        print("No job URLs found to scrape.")
        return

    print(f"Scraping details for {len(job_urls)} jobs...")
    job_details = await linkedin.scrape_jobs(urls=job_urls)

    # Save the detailed job data
    details_output_path = output / "job_details.json"
    with open(details_output_path, "w", encoding="utf-8") as f:
        json.dump(job_details, f, indent=2, ensure_ascii=False)
    print(f"Saved detailed information for {len(job_details)} jobs to {details_output_path}")


if __name__ == "__main__":
    # Make sure to set your SCRAPFLY_KEY environment variable before running
    asyncio.run(main())
```

To run this script, save it and then execute it from your terminal (after setting up the environment as described above):

```shell
poetry run python run_job_scraper.py
```