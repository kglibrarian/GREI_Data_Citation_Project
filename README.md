# GREI outputs cited in Web of Science

This repo holds the code for a cited-reference analysis. It searches the reference lists of Web of Science (WoS) publications for the DOIs of NIH-associated research outputs, such as datasets and software, that GREI repositories registered with DataCite. Every match is recorded with metadata about the citing paper, and citation counts are rolled back up onto each DOI. The results are then summarized as maps, tables and written statements.

The WoS data comes from **ProQuest TDM Studio** exports. These are ZIP files of about 10–15 GB, each holding around 1 million WoS records. The analysis therefore runs **in rolling batches**: download a few exports, process them, delete them and download the next ones. All progress is saved between runs, so no export ever has to be processed twice.

> **Data is not included.** The input DOI list, the WoS exports and all outputs are excluded via `.gitignore`. WoS content is licensed and can't be redistributed.

---

## Quick start

1. Put the DOI list at `data/GREI-DataCite-NIHOnly.csv`. The DOIs must be in a column named `id`.
2. Download one or more WoS exports from TDM Studio into your `Downloads` folder. They are named `wos-data*.zip`, and you should not rename or unzip them.
3. Open **`1_GREI_DataCite_WoS_Cited_Reference_Analysis_v6.ipynb`** and choose **Run All**. New exports are processed, and exports that have already been processed are skipped automatically. Delete the ZIPs once the notebook says it's safe, then repeat with the next batch.
4. When all batches are done, open **`2_GREI_WoS_Citing_Maps_and_Categories.ipynb`** and choose **Run All**. It reads the results of notebook 1 and builds the maps, summary tables, written statements and the results package to share. It takes a few minutes and doesn't need the WoS exports.

Both notebooks are designed to be run top to bottom with **Run All**. Settings are in the first code cell of each notebook.

### Requirements

Python 3.10+ with:

```
pip install pandas numpy lxml openpyxl tqdm plotly kaleido matplotlib
```

You can leave out `lxml`, `tqdm` and `kaleido`. Without them, parsing is slower, progress shows as text instead of a bar, and maps are saved as HTML only instead of HTML and PNG.

---

## Repository layout

```
.
├── 1_GREI_DataCite_WoS_Cited_Reference_Analysis_v6.ipynb   # Step 1: search WoS exports, build the citation list
├── 2_GREI_WoS_Citing_Maps_and_Categories.ipynb             # Step 2: maps, tables, statements, results package
├── README.md
├── .gitignore
├── data/                        # (ignored) input DOI list: GREI-DataCite-NIHOnly.csv
└── GREI_WoS_outputs/            # (ignored) everything the notebooks write — see below
```

Optional helpers, if included:

- `GREI_WoS_export_tracker.csv` lists the TDM Studio exports to request (four-month chunks, 2022–2026), with a place to record each one's status.
- `wos_export_overview.py` and `wos_reference_check.py` are standalone diagnostic scripts. They inspect a single export, showing its record structure, date range and whether reference lists are present, without running the full analysis.

---

## Output folder structure

The layout of `GREI_WoS_outputs/` reflects the order in which the project was built. Notebook 1 writes its files **directly into `GREI_WoS_outputs/`**, plus a few subfolders. Notebook 2 was added later. It writes its working files into **`maps_and_categories/`** and then gathers a curated copy of the most useful results into **`GREI_WoS_results_package/`** and a matching `.zip`.

**If you are a collaborator receiving results, start with `GREI_WoS_results_package/`.** Everything else is working or state files.

```
GREI_WoS_outputs/
│
├── GREI_WoS_results_package/          ← START HERE: the curated package to share (written by notebook 2)
│   ├── 1_GREI_WoS_citations_by_DOI.xlsx       one row per target DOI, with WoS times-cited counts
│   ├── 2_GREI_WoS_citation_list.xlsx          one row per target DOI × citing WoS paper, with citing-paper details
│   ├── 3_GREI_WoS_summary_tables.xlsx         full tables: citing papers, resource types, repositories, years,
│   │                                          document types, countries, US states, US institutions,
│   │                                          organizations, funders, WoS Categories, Research Areas, journals
│   ├── 4_GREI_WoS_data_dictionary.xlsx        every file, tab and column described
│   ├── GREI_WoS_summary_statements.md / .txt  written summaries of the results
│   └── maps_and_charts/                       world and US maps (HTML/PNG) and category charts
├── GREI_WoS_results_package.zip       ← the same package, zipped for sending
│
├── maps_and_categories/               notebook 2 working outputs
│   ├── map_world_citing_countries.html / .png
│   ├── map_us_citing_states.html / .png
│   ├── wos_categories_top.png, research_areas_top.png
│   ├── GREI_WoS_citing_countries_states_categories.xlsx
│   ├── GREI_WoS_us_institutions_and_funding.xlsx   (+ CSV copies)
│   ├── GREI_WoS_citing_paper_us_and_funding_evidence.csv
│   └── GREI_WoS_summary_statements.md / .txt
│
│   ── notebook 1: persistent state (DO NOT DELETE between batches) ──
├── GREI_WoS_processing_log.csv               one row per processed export, with record counts, timings and status
├── GREI_WoS_citation_edges_master.csv        the master citation list: one row per target DOI × citing paper,
│                                             deduplicated across all exports (notebook 2 reads this)
├── GREI_WoS_citation_edges_raw_appendonly.csv  raw matches as found, appended each run (audit trail)
├── GREI_WoS_archive_manifest.csv             each export ZIP, its content fingerprint and the date range it covers
├── GREI_WoS_archive_rename_log.csv           record of any export renaming
├── archive_inventory/                        small per-export date profiles and hashed record-ID lists
│                                             (used for coverage and overlap checks)
├── state_backups/                            rolling backup copies of the state files above
│
│   ── notebook 1: results (rebuilt from the master every run) ──
├── GREI_WoS_citation_edges_master.xlsx       Excel copy of the master citation list
├── GREI_WoS_target_aggregates.csv            citation counts per target DOI
├── GREI-DataCite-NIHOnly_with_WoS_citations.xlsx / .csv   the input DOI list with WoS counts added
├── GREI_WoS_vs_DataCite_citation_comparison.csv           WoS counts next to DataCite's own citation counts
├── GREI_WoS_unmatched_target_dois.csv        target DOIs with no WoS citation found so far
├── summary_tables/GREI_WoS_summary_tables.xlsx            first-pass summary tables
├── coverage_report/                          which months and years the processed exports cover, and any gaps
│                                             (CSV files plus GREI_WoS_coverage_report.xlsx)
│
│   ── notebook 1: diagnostics ──
├── GREI_WoS_malformed_cited_ref_doi_samples.csv   examples of unusable DOIs in reference lists
├── GREI_WoS_possible_version_near_misses.csv      references to a different version of a target DOI
└── wos_schema_profile_sample.csv                  record-structure profile from a sample export
```

Files beginning with `TEST_` come from test-mode runs (`TEST_MODE = True`) and can be deleted.

**Between batches, keep the state files:** the processing log, the master citation list, the manifest, `archive_inventory/` and `state_backups/`. They record which exports have already been processed. Exports are identified by a fingerprint of their contents, so it doesn't matter that every TDM Studio download has the same generic file name. Everything else can be regenerated with Run All.

---

## Method notes

- **Matching.** A citation is counted when a target DOI appears, after normalization, in the cited-reference list of a WoS record. The record's original Clarivate XML is unwrapped from the ProQuest TDM Studio wrapper, and DOIs are read from `static_data/fullrecord_metadata/references/reference`. A fast byte-level prefilter skips records that contain none of the target DOI prefixes. This doesn't change the results.
- **Counting.** Counts are of distinct citing publications unless stated otherwise. For countries, states and institutions, a paper is counted once for each place in its author addresses (whole counting).
- **Funding.** NIH and US-federal acknowledgment is identified from WoS funding fields and grant-number patterns.
- **Conservative counts.** Data citations without a DOI, citations only in data-availability statements and citations in publications outside WoS are not detected, so these counts will differ from DataCite's.
- **Coverage.** The analysis covers WoS publications from January 2022 onward. TDM Studio coverage of the most recent months is thinner. See `coverage_report/`.
- **Not included.** Normalized metrics, such as citing-journal quartiles or impact factors and citing-paper percentiles, are not calculated. Citing papers keep their WoS accession numbers (UT), so these can be added later, for example from InCites.

---

## Credits

Analysis and code by Karen Gutzman (Galter Health Sciences Library, Northwestern University Feinberg School of Medicine) for Keith Herzog and Veronica Hoyo, as part of work on the NIH Generalist Repository Ecosystem Initiative (GREI). Web of Science data was accessed through ProQuest TDM Studio under Northwestern's license.
