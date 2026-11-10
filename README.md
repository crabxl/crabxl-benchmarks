# CrabXL benchmarks

Speed and peak memory of CrabXL compared with other XLSX libraries.

| Suite | Compares |
|---|---|
| [Rust](rust/README.md) | crabxl, calamine, rust_xlsxwriter, umya-spreadsheet, oxlsx |
| [Python](python/README.md) | crabxl, wolfxl, oxlsx, python-calamine, fastexcel, XlsxWriter, openpyxl |

Clone [crabxl](https://github.com/crabxl/crabxl) next to this repository. The
Rust driver builds against `../crabxl`. Run the Rust suite first: the Python
suite reads its xl, xxl and nyc311 files from `rust/data/`.

Run the suites on an idle computer that is connected to power. Other programs
change the results.
