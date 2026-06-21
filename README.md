# 🛡️ Threat Intelligence Enrichment Tool

A Python-based CLI tool that automatically enriches domains, IP addresses, and URLs by querying multiple open-source and commercial threat intelligence APIs — consolidating WHOIS, reputation, and abuse data into a single structured JSON report.

> This is a command-line tool — there's no hosted demo/website for it, just clone and run.

---

## 📌 Overview

Security analysts often need to pull context on an indicator of compromise (IP, domain, or URL) from several different sources before deciding whether it's malicious. Doing this manually across multiple dashboards is slow. This tool automates that workflow:

- Accepts a single indicator, a batch file of indicators, or a `key:value` pair from the command line.
- Automatically detects the indicator type (domain / IP address / URL).
- Queries the relevant intelligence sources for that type in parallel.
- Returns a single, consolidated JSON report — printed to stdout or written to a file.
- Supports multithreaded batch processing for large indicator lists.

---

## ✨ Key Features

- 🔍 **Multi-Source Enrichment** — Aggregates data from:
  - **WHOIS** — domain/IP registration details (for domains and IPs)
  - **VirusTotal** — reputation, detection stats, and category data (for domains, IPs, and URLs)
  - **AbuseIPDB** — abuse confidence score and reported activity (for IPs)
  - **IPinfo** — geolocation and network ownership details (for IPs)
- 🧠 **Automatic Indicator Parsing** — Detects whether an input is a domain, IP, or URL and routes it to the right set of lookups via `scanner/parser.py` and `scanner/manager.py`.
- ⚡ **Multithreaded Batch Processing** — Process large indicator files in parallel using the `-t/--thread` flag, splitting work across multiple processes (`scanner/multithreading.py`).
- 📄 **Flexible Input Modes**:
  - Single indicator (`-c`)
  - Batch file of indicators (`-f`)
  - Explicit key:value pair (`-A`, e.g. `domain:google.com`)
- 💾 **JSON Output** — Print results to the console or save them to a file with `-o`.
- 🧩 **Modular Design** — Each intelligence source lives in its own module under `scanner/`, making it easy to add new providers.

---

## 🛠️ Tech Stack

| Category | Tools |
|---|---|
| Language | Python 3 |
| CLI | argparse |
| Concurrency | multiprocessing |
| WHOIS | python-whois |
| Threat Intel APIs | vt-py (VirusTotal), AbuseIPDB REST API, IPinfo |
| HTTP | requests |
| Networking | scapy |

---

## 📂 Project Structure

```
Threat-Intelligence-Enrichment-Tool/
├── docs/                     # Sample test and output files
├── scanner/                  # Core modules
│   ├── __init__.py
│   ├── parser.py             # Parses CLI/file input into (type, value) indicators
│   ├── manager.py            # Routes each indicator to the right lookup modules
│   ├── multithreading.py     # Parallel batch-processing logic
│   ├── whois_lookup.py       # WHOIS enrichment
│   ├── virustotal.py         # VirusTotal enrichment
│   ├── abuseipdb.py          # AbuseIPDB enrichment
│   ├── ipinfo_lookup.py      # IPinfo enrichment
│   └── shodanmod.py          # Shodan integration (in progress)
├── main.py                   # CLI entry point
├── requirements.txt          # Dependency list
├── setup.py                  # Packaging script (installs as `scanner` console command)
└── README.md
```

---

## ⚙️ How It Works

1. **Parse input** — `main.py` accepts a single value (`-c`), a file of values (`-f`), or an explicit `key:value` pair (`-A`). `scanner/parser.py` classifies each value as a `domain`, `ipaddress`, or `url`.
2. **Route to enrichment sources** — `scanner/manager.py`'s `Manager.controller()` decides which lookups to run per indicator type:
   - `domain` / `ipaddress` → WHOIS
   - `ipaddress` → AbuseIPDB + IPinfo
   - `domain` / `ipaddress` / `url` → VirusTotal
3. **(Optional) Parallelize** — If `-t` is provided alongside `-f`, the input file is split into chunks and processed concurrently via `multiprocessing`, with each worker writing partial results that are merged at the end.
4. **Output** — Combined results are serialized to JSON and either printed to the console or written to the path given with `-o`.

---

## 🚀 Getting Started

### Prerequisites
- Python 3.8+
- API keys for [VirusTotal](https://www.virustotal.com/gui/join-us), [AbuseIPDB](https://www.abuseipdb.com/account/api), and [IPinfo](https://ipinfo.io/signup)

### Installation

```bash
# Clone the repository
git clone https://github.com/KoushikVatam/Threat-Intelligence-Enrichment-Tool.git
cd Threat-Intelligence-Enrichment-Tool

# Create a virtual environment (recommended)
python3 -m venv venv
source venv/bin/activate   # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Configuration

Before running, add your API keys to the relevant modules:

```python
# scanner/abuseipdb.py
api_key = "YOUR_ABUSEIPDB_API_KEY"

# scanner/virustotal.py
vt.Client("YOUR_VIRUSTOTAL_API_KEY")

# scanner/ipinfo_lookup.py
access_token = "YOUR_IPINFO_ACCESS_TOKEN"
```

> ⚠️ For production use, move these keys into environment variables instead of hardcoding them in source files.

### Usage

```bash
# Look up a single domain
python3 main.py -c google.com

# Look up an indicator from a key:value pair
python3 main.py -A domain:google.com

# Process a file of indicators
python3 main.py -f /path/to/input.txt

# Process a file with multithreading and save output to a file
python3 main.py -f /path/to/input.txt -t 4 -o /path/to/output.json

# Save a single lookup result to a file
python3 main.py -c 8.8.8.8 -o /path/to/output.json
```

**CLI Options**

| Flag | Description |
|---|---|
| `-c, --custom` | Provide a single domain, IP, or URL |
| `-f, --file` | Provide a path to a text file of indicators |
| `-A, --api` | Provide a `key:value` pair (e.g. `domain:google.com`) |
| `-t, --thread` | Number of threads/processes for batch file processing |
| `-o, --output` | Path to write the JSON report (defaults to stdout) |

---

## 🔮 Future Improvements

- Wire up the in-progress `scanner/shodanmod.py` module into `Manager.controller()` for Shodan-based port/service enrichment.
- Move API keys to environment variables / a `.env` file instead of hardcoding them in module source.
- Add automated tests for the parser, manager routing logic, and each enrichment module.
- Add rate-limit handling/backoff for the free tiers of VirusTotal, AbuseIPDB, and IPinfo.
- Add an output formatter (CSV/table) as an alternative to raw JSON.

---

## 📄 License

No license file is currently included in this repository — add a `LICENSE` file to formally specify usage terms.

---

## 👤 Author

**Koushik Vatam**
GitHub: [@KoushikVatam](https://github.com/KoushikVatam)
