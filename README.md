# mzParquet Converter

Convert mass spectrometry vendor files to the open [`.mzparquet`](https://github.com/mzparquet) format.

Download all binaries and installers from the [Releases page](https://github.com/ChaparralLabs/mzparquet_converter/releases).

For full documentation see the [mzParquet Converter manual](https://manual.chaparral.ai/command_line_tools/mzparquet_converter/).

## GUI Application

A desktop application for drag-and-drop conversion without the command line.

| Platform | Download |
|----------|----------|
| Windows x64 (.msi) | [mzParquet Converter Installer](https://github.com/ChaparralLabs/mzparquet_converter/releases/download/latest/mzParquet.Converter_0.1.0_x64_en-US.msi) |
| Windows x64 (.exe) | [mzParquet Converter Setup](https://github.com/ChaparralLabs/mzparquet_converter/releases/download/latest/mzParquet.Converter_0.1.0_x64-setup.exe) |
| macOS Apple Silicon (.dmg) | [mzParquet Converter](https://github.com/ChaparralLabs/mzparquet_converter/releases/download/latest/mzParquet.Converter_0.1.0_aarch64.dmg) |

| Platform | Thermo `.raw` | Bruker `.d` |
|----------|:-------------:|:-----------:|
| Windows  | ✅ | ✅ |
| macOS (Apple Silicon) | ❌ | ✅ |

Features:
- **Drag and drop** `.raw` files or `.d` folders
- **Batch conversion** with real-time progress tracking
- **Skip existing** outputs (with option to force re-convert)

> **Note:** Thermo `.raw` conversion is not available on macOS due to ThermoFisher library limitations. Linux users: use the CLI tools below.

## CLI Downloads

### ThermoParquet — Thermo `.raw` → `.mzparquet`

| Platform | Download |
|----------|----------|
| Windows x64 | [ThermoParquet-x86_64-pc-windows-msvc.exe](https://github.com/ChaparralLabs/mzparquet_converter/releases/download/latest/ThermoParquet-x86_64-pc-windows-msvc.exe) |
| Linux x64 | [ThermoParquet-x86_64-unknown-linux-gnu](https://github.com/ChaparralLabs/mzparquet_converter/releases/download/latest/ThermoParquet-x86_64-unknown-linux-gnu) |

> **Note:** ThermoParquet requires the ThermoFisher RawFileReader runtime, which is only available on Windows and Linux.

### dotD2parquet — Bruker `.d` → `.mzparquet`

| Platform | Download |
|----------|----------|
| Windows x64 | [dotD2parquet-x86_64-pc-windows-msvc.exe](https://github.com/ChaparralLabs/mzparquet_converter/releases/download/latest/dotD2parquet-x86_64-pc-windows-msvc.exe) |
| Linux x64 | [dotD2parquet-x86_64-unknown-linux-gnu](https://github.com/ChaparralLabs/mzparquet_converter/releases/download/latest/dotD2parquet-x86_64-unknown-linux-gnu) |
| macOS Apple Silicon | [dotD2parquet-aarch64-apple-darwin](https://github.com/ChaparralLabs/mzparquet_converter/releases/download/latest/dotD2parquet-aarch64-apple-darwin) |

## Usage

```bash
# Thermo .raw → .mzparquet
ThermoParquet sample.raw
# Output: sample.mzparquet

# Bruker .d → .mzparquet
dotD2parquet sample.d
# Output: sample.mzparquet
```

#### Batch conversion (shell)

```bash
# Convert all .raw files in a directory
for f in *.raw; do ThermoParquet "$f"; done

# Convert all .d folders
for d in *.d; do dotD2parquet "$d"; done
```

#### Batch conversion (PowerShell)

```powershell
# Convert all .raw files
Get-ChildItem *.raw | ForEach-Object { .\ThermoParquet.exe $_.FullName }

# Convert all .d folders
Get-ChildItem -Directory *.d | ForEach-Object { .\dotD2parquet.exe $_.FullName }
```

## Output Format

The `.mzparquet` file is an Apache Parquet file with Zstandard compression and the following schema:

| Column | Type | Description |
|--------|------|-------------|
| `scan` | `uint32` | Scan/spectrum number |
| `level` | `uint32` | MS level (1, 2, …) |
| `rt` | `float32` | Retention time (minutes for Thermo, seconds for Bruker) |
| `mz` | `float32` | Mass-to-charge ratio |
| `intensity` | `uint32` | Peak intensity |
| `ion_mobility` | `float32?` | Ion mobility (1/K0, Bruker only) |
| `isolation_lower` | `float32?` | Isolation window lower bound |
| `isolation_upper` | `float32?` | Isolation window upper bound |
| `precursor_scan` | `uint32?` | Parent MS1 scan number |
| `precursor_mz` | `float32?` | Precursor m/z |
| `precursor_charge` | `uint32?` | Precursor charge state |

Files can be read with any Parquet library — Python (pyarrow, polars, pandas), R (arrow), Rust, etc.

```python
import polars as pl

df = pl.read_parquet("sample.mzparquet")
print(df.head())
```

## Supported Instruments

| Vendor | Format | Converter | Platforms |
|--------|--------|-----------|-----------|
| Thermo Fisher | `.raw` | ThermoParquet | Windows, Linux |
| Bruker timsTOF | `.d` | dotD2parquet | Windows, Linux, macOS |

## Linux Compatibility

CLI binaries are built on Ubuntu 22.04 (glibc 2.35). Compatible distros:

| Distro | glibc | Compatible |
|--------|-------|:----------:|
| Ubuntu 22.04 LTS | 2.35 | ✅ |
| Ubuntu 24.04 LTS | 2.39 | ✅ |
| Debian 12 (Bookworm) | 2.36 | ✅ |
| Fedora 36+ | 2.35+ | ✅ |
| Rocky / Alma Linux 9 | 2.34 | ❌ |
| Ubuntu 20.04 LTS | 2.31 | ❌ |
| CentOS 7 | 2.17 | ❌ |

> **Requires glibc ≥ 2.35.** Check your version: `ldd --version`
>
> ThermoParquet additionally requires `libicu` and `openssl` (pre-installed on most Ubuntu/Debian systems).

## License

Copyright © Chaparral Labs. All rights reserved.
