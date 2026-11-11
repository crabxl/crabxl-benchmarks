# CrabXL benchmarks

Speed and peak memory of [CrabXL for Python](https://pypi.org/project/crabxl/)
compared with other Python XLSX libraries. See the [results](results/results.md).

## Run

Install [uv](https://docs.astral.sh/uv/), then run:

```sh
uv run run.py
```

uv installs CrabXL and every compared library from PyPI. The suite writes its
input files to `data/` (ignored by Git) on the first run and reuses them after
that. Build the nyc311 file by hand (see below). Results go to `results/`. Run the suite on an
idle computer that is connected to power.

```sh
uv run run.py --sizes small --runs 1 --libraries crabxl openpyxl
uv run run.py --sizes nyc311
uv run run.py --report-only
```

## Scenarios

| Scenario | Task | Libraries |
|---|---|---|
| Read | Open the file and visit every cell value. | crabxl, wolfxl, oxlsx, python-calamine (`iter_rows` and `to_python`), fastexcel, openpyxl |
| Write | Create the sheet with `append` (or `write_row`) and save it. | crabxl, wolfxl, oxlsx, XlsxWriter, openpyxl |
| Edit | Load the file, change one cell in every 100 rows, append 100 rows and save. | crabxl, wolfxl (`modify=True`), oxlsx, openpyxl |

crabxl, wolfxl and openpyxl run in their normal mode and in their `read_only`
or `write_only` mode. XlsxWriter runs in normal and `constant_memory` mode.
python-calamine and fastexcel only read, and XlsxWriter only writes. openpyxl
is slow, so it runs on the small size only. The nyc311 file is read only, by
crabxl, wolfxl, python-calamine and fastexcel.

## Data

XlsxWriter generates the input, so no compared reader reads its own output.
Each row has 10 cells: an integer, a decimal number, a repeated text (1,000
different values), a unique text, a date, a boolean and four decimal numbers.

| Size | Rows × columns | Cells | File |
|---|---:|---:|---:|
| small | 10,000 × 10 | 100,000 | 0.6 MB |
| medium | 100,000 × 10 | 1,000,000 | 6 MB |
| nyc311 | 1,000,001 × 41 | 28,056,975 non-empty | 220 MB |

### NYC 311 file

The nyc311 size is the real data set that the
[calamine README](https://github.com/tafia/calamine#performance) uses: one
million New York City 311 service requests with 41 columns, mostly text, with
empty fields. To build it:

1. Download
   [NYC_311_SR_2010-2020-sample-1M.7z](https://raw.githubusercontent.com/wiki/jqnatividad/qsv/files/NYC_311_SR_2010-2020-sample-1M.7z)
   (48 MB) and extract the CSV.
2. Open the CSV in Excel and save it as `data/nyc311.xlsx`.

Readers count non-empty cells only, because libraries return empty cells
differently. All readers read the first worksheet.

## Method

- Each case runs in a new Python process. Time is the wall time of the whole
  process, including interpreter start-up and imports. Memory is the peak
  resident memory of the process (`PeakWorkingSetSize` on Windows,
  `ru_maxrss` on Linux and macOS). The tables also show the CPU time (user
  plus kernel) of the process.
- On Windows, the suite starts the base interpreter directly with the virtual
  environment on `PYTHONPATH`. The `python.exe` of a virtual environment is a
  launcher process, so its memory is not the memory of the case.
- Each case runs one time to warm the file cache, then three more times. The
  tables show the median of the three. A case whose first run takes more than
  10 seconds is measured by that run only.
- Every case runs single-threaded: the process starts suspended and is pinned
  to the same single logical CPU for every library before it runs any code.
- wolfxl's safety limits, which reject large files, are raised.
- Every case prints the number of cells it handled. Check `cells` in
  `results.json` to confirm that all libraries did the same work.
- A case that takes more than 2 minutes is shown as a timeout. A case that
  takes more than 12 seconds is not run at the next size (10 times more rows)
  and is shown as an estimated timeout. A case that raises an error is shown
  as failed.
