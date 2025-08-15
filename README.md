# Road Accident Data Extraction Tool (GTR — Government Table Reader)

*A Python tool to automatically extract, clean, and structure tabular data from India’s MoRTH “Road Accidents in India” annual PDF reports.*

---

## Table of Contents

* [Overview](#overview)
* [Features](#features)
* [Technical Stack](#technical-stack)
* [Installation](#installation)

  * [Prerequisites](#prerequisites)
  * [Python Dependencies](#python-dependencies)
* [Usage](#usage)

  * [Basic Usage](#basic-usage)
  * [Key Functions](#key-functions)
  * [PDF Batch Processing](#pdf-batch-processing)
* [Data Sources](#data-sources)
* [Output Structure](#output-structure)

  * [CSV Files](#csv-files)
  * [Metadata Files](#metadata-files)
  * [Example Output Tree](#example-output-tree)
* [Key Functions Explained](#key-functions-explained)
* [Error Handling & Quality Control](#error-handling--quality-control)
* [Performance Considerations](#performance-considerations)
* [Limitations](#limitations)
* [Contributing](#contributing)
* [License](#license)
* [Support & Troubleshooting](#support--troubleshooting)
* [Acknowledgments](#acknowledgments)

---

## Overview

This project automates extraction of statistical tables from PDF documents published by the **Ministry of Road Transport and Highways (MoRTH), Government of India**. It leverages OCR and table-detection to convert PDF tables into structured **CSV** (plus rich metadata), preserving data integrity and table semantics (including multi-row headers).

## Features

* **Automated PDF Processing:** Download and process multiple years of Road Accident reports *(2008–2022)*.
* **Intelligent Table Detection:** Uses `img2table` with `Tesseract OCR` for accurate table extraction.
* **Multi-row Header Handling:** Detects and combines complex multi-row headers.
* **Data Cleaning:** Standardizes columns, datatypes, and numerics; handles merged/empty cells.
* **Metadata Generation:** Creates comprehensive metadata sidecar files for each table.
* **Batch Processing:** Processes multiple PDFs sequentially to manage memory.
* **Quality Control:** Validation checks for completeness and accuracy; skips empty tables.

## Technical Stack

* **Language:** Python 3.x
* **Libraries:**

  * `OpenCV` — image processing for table detection
  * `img2table` — table extraction from PDFs/images
  * `pytesseract`/`Tesseract OCR` — OCR engine
  * `pandas` — data wrangling
  * `Pillow (PIL)` — image I/O
  * `requests` — file download helper
  * `numpy` — numerics & array ops
  * `xlsxwriter` — Excel export (optional)

---

## Installation

### Prerequisites

Install **Tesseract OCR** on your system.

```bash
# Ubuntu/Debian
sudo apt-get update && sudo apt-get install -y tesseract-ocr

# macOS (Homebrew)
brew install tesseract

# Windows
# Download & install from: https://github.com/UB-Mannheim/tesseract/wiki
```

Install system libs (Linux):

```bash
sudo apt-get update
sudo apt-get install -y libgl1-mesa-glx
```

### Python Dependencies

Install via `pip` (preferably in a virtual environment):

```bash
pip install opencv-contrib-python img2table pandas pillow requests numpy xlsxwriter
```

> **Tip:** If you also use `pytesseract`, ensure the Tesseract binary is on PATH or set `pytesseract.pytesseract.tesseract_cmd` explicitly on Windows.

---

## Usage

### Basic Usage

1. **Clone or download** this repository / notebook.
2. **Install** dependencies as above.
3. **Run** the notebook or Python scripts in sequence.

### Key Functions

```python
# Process an extracted DataFrame with header detection & cleaning
df_cleaned = process_img2table_df(df)

# Combine multi-row headers (e.g., first 2 rows are header rows)
df_clean = combine_multirow_headers_clean(df, num_header_rows=2)

# Detect primary identifier (e.g., State/UT names, vehicle types)
row_id_column = detect_row_id_column(df)
```

### PDF Batch Processing

```python
# Process all PDFs in a folder and write outputs to Extracted_Tables/
process_all_pdfs_in_folder("/path/to/pdfs", output_dir="Extracted_Tables")
```

---

## Data Sources

* **Base URL:** `https://morth.nic.in/sites/default/files/`
* **Years Covered:** *2008–2022*
* **Report Format:** Annual statistical PDF reports (*Road Accidents in India* series)

> The tool can auto-download PDFs given the base URL and year list, or you may place files locally.

---

## Output Structure

### CSV Files

Each table is saved with a descriptive filename:

```
Page_{page_number}_Table_{table_index}_{table_title}.csv
```

### Metadata Files

For every CSV, a matching `.txt` sidecar includes:

* Table title and source
* Column headers and inferred data types
* Row/column counts
* Data quality metrics
* Quick statistics (min, max, totals)

### Example Output Tree

```
Extracted_Tables/
├── Page_20_Table_1_Total_Accidents_Fatalities_2018_2022.csv
├── Page_20_Table_1_Total_Accidents_Fatalities_2018_2022.txt
├── Page_43_Table_1_Fatal_Accidents_by_Road_Categories.csv
├── Page_43_Table_1_Fatal_Accidents_by_Road_Categories.txt
└── ...
```

---

## Key Functions Explained

### `detect_row_id_column(df, threshold=0.8)`

Identifies the likely primary identifier column (e.g., *State/UT*, *Vehicle Type*). Uses heuristics on text density, uniqueness, and categorical stability; `threshold` tunes the confidence cutoff.

### `combine_multirow_headers_clean(df, num_header_rows=2)`

Merges multi-row headers into single, meaningful column names while resolving duplicates and empty cells (e.g., `"2019"` under `"Fatalities"` → `"Fatalities__2019"`). Handles whitespace and punctuation normalization.

### `guess_header_rows(df, max_rows=10)`

Infers how many top rows belong to the header by scanning for numeric ratios, repeated tokens, and alignment cues up to `max_rows`.

### `process_img2table_df(df)`

End‑to‑end cleaner that:

* Detects header structure (may call `guess_header_rows`)
* Combines multi-row headers
* Normalizes column names
* Handles merged/empty cells & footnotes
* Standardizes numeric formats and dtypes

---

## Error Handling & Quality Control

* **Corrupt PDF Detection:** Validate PDFs before parsing.
* **Empty Table Filtering:** Skip tables with >90% empty cells.
* **Memory Management:** Process one PDF at a time.
* **Exception Logging:** Detailed logs for reproducible debugging.

---

## Performance Considerations

* **Memory:** Sequential PDF processing to cap RAM usage.
* **Time:** Large PDFs can take minutes; OCR and table detection are compute-heavy.
* **Storage:** Many tables → many CSV/metadata files; plan disk usage accordingly.

---

## Limitations

* **PDF Quality:** Works best with clear, machine-readable PDFs.
* **Table Format:** Optimized for structured, bordered tables.
* **Language:** Tuned for English OCR.
* **Complex Layouts:** Very complex or nested tables may need manual review.

---

## Contributing

1. **Fork** the repository.
2. **Create** a feature branch.
3. **Commit** your changes (add tests where possible).
4. **Open** a pull request with a clear description and context.

---

## License

This project is intended for academic and research purposes focused on analysis of Indian government statistical data.

---

## Support & Troubleshooting

* Review the **error logs** printed during runs.
* Verify all **dependencies** (including system-level Tesseract install).
* Ensure **PDF paths/URLs** are correct and files are not corrupted.
* Check the function **docstrings** for parameter expectations and return types.

> If issues persist, please open an issue with a minimal reproducible example (PDF page, code snippet, and logs).

---

## Acknowledgments

* **MoRTH** for providing open access to the *Road Accidents in India* statistical reports.
* **img2table** developers for robust table extraction capabilities.
* **Tesseract OCR** community for a reliable OCR engine.
