## Bihar Electoral Rolls 2020

[![License: MIT](https://img.shields.io/badge/code-MIT-green)](LICENSE)

> **Archived.** The scripts scraped a site at one point in time and are kept as a record of how the data were produced.

Every primary electoral-roll PDF the Chief Electoral Officer, Bihar published at `http://ele.bihar.gov.in/pdfsearch/` (publication date 2020-02-07): 72,723 parts (one PDF per polling-station part) across all 243 assembly constituencies, 284 GiB in total, downloaded 2020-07-20 to 2020-08-06. This repository holds the scripts and an index of every file. The PDFs are in a Google Cloud Storage bucket, not on Dataverse; because the rolls carry personal information, access is restricted to research use. Part of the [Indian electoral rolls](https://github.com/in-rolls/electoral_rolls) collection.

### Data

| File | Where | Rows | What |
|---|---|---|---|
| `FinalRoll_ACNo_<AC>PartNo_<Part>.pdf` | `gs://in-electoral-rolls-2020/bihar/` | 72,723 files | the rolls, 109 KB to 6.7 MB each (median 4.2 MB) |
| [`metadata_and_log_files/bihar.csv`](metadata_and_log_files/bihar.csv) | this repository | 72,723 | **use this one**: one row per PDF, with constituency and polling-station names, bucket path, size, upload time |
| [`metadata_and_log_files/metadata.csv`](metadata_and_log_files/metadata.csv) | this repository | 72,723 | the two dropdown labels per part, as read from the site; the name columns of `bihar.csv` |
| [`metadata_and_log_files/list.txt`](metadata_and_log_files/list.txt) | this repository | 72,204 | bucket listing after the first download pass |
| [`metadata_and_log_files/list2.txt`](metadata_and_log_files/list2.txt) | this repository | 72,723 | bucket listing after the patch pass |
| [`metadata_and_log_files/list3.txt`](metadata_and_log_files/list3.txt) | this repository | 72,723 | `gsutil ls -al` of the final bucket, plus a total line |

The first pass missed 519 parts; `list2.txt` minus `list.txt` is exactly the set the patch script fetched.

**Columns** of `bihar.csv`:

| Column | Notes |
|---|---|
| `ac_no` | Assembly constituency number, 1--243, parsed from the filename. |
| `part_no` | Part number within the constituency, 1--408, parsed from the filename. `(ac_no, part_no)` is unique. |
| `assembly_segment` | Constituency label from the site's dropdown, in Hindi, e.g. `1 - वाल्मीकिनगर`. |
| `part_number` | Part label from the site's dropdown: number and polling-station name, e.g. `1 - उत्क्रमित मध्य विद्यालय, ठाढ़ी`. |
| `pdf_path` | Bucket path with the GCS generation suffix, e.g. `gs://in-electoral-rolls-2020/bihar/FinalRoll_ACNo_1PartNo_1.pdf#1596200269282980`. Drop everything from `#` to get the object name. |
| `filesize` | Bytes. |
| `timestamp` | Upload time to the bucket, UTC. |

`bihar.csv` was built by sorting the bucket listing by `(ac_no, part_no)` and joining `metadata.csv` to it row by row, not on a key. The join lines up: in all 72,723 rows the leading number of `assembly_segment` equals `ac_no` and the leading number of `part_number` equals `part_no`.

### How the data were collected

1. [`scripts/scrape-dry-run-getmeta.ipynb`](scripts/scrape-dry-run-getmeta.ipynb) drives the site in headless Chrome (Selenium), walks both dropdowns and saves every constituency/part label to `metadata.csv`. This is the reference list of what should exist.
2. [`scripts/bihar.py`](scripts/bihar.py) walks the same dropdowns, selects the primary roll, screenshots the CAPTCHA, solves it through anti-captcha, submits, and uploads each constituency's downloads to the bucket. Arguments `[constituency index] [part index]` resume a crashed run.
3. [`scripts/bihar_check.ipynb`](scripts/bihar_check.ipynb) compares the bucket listing to the dry-run counts and prints the missing parts; [`scripts/bihar_patch.py`](scripts/bihar_patch.py) fetches exactly those.
4. [`scripts/bihar_to_csv.ipynb`](scripts/bihar_to_csv.ipynb) parses `list3.txt` and joins the labels to produce `bihar.csv`.

### Getting the PDFs

Request access through the procedure described in [in-rolls/electoral_rolls](https://github.com/in-rolls/electoral_rolls#how-do-i-get-the-electoral-roll-pdfs) (a form plus IRB approval). Once granted, you get read access to the bucket for a month. The bucket is requester-pays and in the Archive storage class, so reads are billed to a Google Cloud project of yours:

```bash
gsutil -u your-billing-project ls gs://in-electoral-rolls-2020/bihar/
gsutil -u your-billing-project cp gs://in-electoral-rolls-2020/bihar/FinalRoll_ACNo_1PartNo_1.pdf .
```

### Citation

See [CITATION.cff](CITATION.cff).

### License

Code is MIT. The rolls are published by the Chief Electoral Officer, Bihar; the bucket copy is restricted to research use.
