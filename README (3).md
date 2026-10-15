# VulnScanner

A web application vulnerability scanner built for Kali Linux penetration testers. Point it at a domain through the web UI and it runs a 19-stage recon-to-exploitation pipeline, chaining dozens of well-known offensive security tools together.

Flask backend + HTML/JS frontend; the actual scanning logic lives in a set of bash scripts under `scripts/`.

> Internal pipeline version: v13 (Stage 11 rewritten — see "Pipeline notes" below).

## Stack

- **Backend**: Python / Flask (`app.py`)
- **Frontend**: Pure HTML/JS (`templates/index.html`)
- **Scripts**: Bash (`scripts/`)

## Setup

```bash
pip install flask
python app.py
# open http://localhost:5000
```

Scan results are written to `outputs/<domain>_<timestamp>/`. Browse and download them from the **Outputs** tab in the UI.

## Tool installation

The scanner shells out to a large number of external tools, which need to be installed and on `$PATH` first:

```bash
# Go tools
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
go install -v github.com/projectdiscovery/katana/cmd/katana@latest
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
go install github.com/lc/gau/v2/cmd/gau@latest
go install github.com/hahwul/dalfox/v2@latest
go install github.com/tomnomnom/gf@latest
go install github.com/tomnomnom/unfurl@latest
go install github.com/haccer/subjack@latest
go install github.com/gitleaks/gitleaks/v8@latest

# System tools
sudo apt install nmap whatweb nikto wapiti sqlmap ffuf

# Python tools
pip install uro
python3 -m venv ~/venv
~/venv/bin/pip install paramspider arjun

# Python clones
git clone https://github.com/GerbenJavado/LinkFinder ~/tools/LinkFinder
pip install -r ~/tools/LinkFinder/requirements.txt

git clone https://github.com/m4ll0k/SecretFinder ~/tools/SecretFinder
pip install -r ~/tools/SecretFinder/requirements.txt

# gf patterns
mkdir -p ~/.gf
git clone https://github.com/tomnomnom/gf ~/tools/gf
cp ~/tools/gf/examples/*.json ~/.gf/
git clone https://github.com/1ndianl33t/Gf-Patterns ~/tools/Gf-Patterns
cp ~/tools/Gf-Patterns/*.json ~/.gf/
```

## Pipeline (19 stages)

| # | Script | Tool(s) | Key outputs |
|---|--------|---------|-------------|
| 01 | `01_subfinder.sh` | Subfinder | `subdomains.txt` |
| 02 | `02_httpx.sh` | httpx | `live.txt` |
| 03 | `03_whatweb.sh` | WhatWeb | `whatweb.txt` |
| 04 | `04_nmap.sh` | Nmap | `nmap.txt` |
| 05 | `05_subjack.sh` | Subjack | `subjack.txt` |
| 06 | `06_url_collection.sh` | GAU + Katana | `gau.txt`, `katana.txt`, `allurls.txt` |
| 07 | `07_url_filter.sh` | uro + unfurl | `filtered_urls.txt`, `paths.txt`, `params_keys.txt` |
| 09 | `09_js_extraction.sh` | Katana-jc + LinkFinder + SecretFinder + Gitleaks | `jsfiles.txt`, `js_endpoints.txt`, `js_secrets.txt`, `gitleaks.txt` |
| 10 | `10_param_discovery.sh` | ParamSpider + Arjun | `paramspider.txt`, `arjun_params.txt`, `total_params.txt` |
| 11 | `11_paramurls.sh` | ffuf + httpx + gf | `urls_with_params.txt`, `urls_without_params.txt`, `total_endpoints.txt`, `paramurls1.txt`, `paramurls2.txt`, `final_paramurls.txt`, `gf_*.txt` |
| 12 | `12_nuclei.sh` | Nuclei | `nuclei.txt` |
| 13 | `13_dalfox_sqlmap.sh` | Dalfox + SQLMap | `dalfox.txt`, `sqlmap.txt` |
| 14 | `14_wapiti_nikto.sh` | Wapiti + Nikto | `wapiti.txt`, `nikto.txt` |
| 15 | `15_cors.sh` | CORScanner | `cors.txt` |
| 16 | `16_cloud_enum.sh` | cloud_enum + S3Scanner | `cloud_enum.txt`, `s3scanner.txt` |
| 17 | `17_gitdorker.sh` | GitDorker | `gitdorker.txt` |
| 18 | `18_race.sh` | race-the-web | `race.txt` |
| 19 | `19_waf.sh` | wafw00f | `waf.txt` |

### Pipeline notes (v13, Stage 11)

- `urls_with_params.txt` is now built from `filtered_urls.txt` (param URLs) + `paramspider.txt` only (no longer includes `params_keys`).
- `urls_without_params.txt` is built from `filtered_urls.txt` (clean URLs only).
- `total_endpoints.txt` = ffuf dir-bruteforce paths + `js_endpoints.txt` (LinkFinder) + `paths.txt` (unfurl).
- The param fuzzing wordlist (SecLists `burp-parameter-names` + `arjun_params.txt` + `params_keys.txt`) is combined and deduplicated directly in Stage 11, no longer relying on `total_params.txt` from Stage 10.

**Stage 11 detail:**

```
PRE-STEP (after paramspider/arjun in stage 10)
  filtered_urls (has params) + paramspider.txt  →  urls_with_params.txt
  filtered_urls (no params)                     →  urls_without_params.txt

PARAM WORDLIST (used in all fuzzing):
  SecLists burp-parameter-names + total_params.txt (arjun_params + unfurl params_keys)
  → combined + deduped → param_wl

PIPELINE 1  urls_with_params.txt
  → live filter → value fuzzing (ffuf) → paramurls1.txt

PIPELINE 2  urls_without_params.txt
  → ffuf dir bruteforce (SecLists) → discovered paths
  → combine: discovered paths + js_endpoints.txt + paths.txt → total_endpoints.txt
  → build full URLs (live hosts × total_endpoints)
  → live filter → param fuzzing (param_wl, ?FUZZ=1) → value fuzzing (ffuf)
  → paramurls2.txt

FINAL MERGE
  paramurls1.txt + paramurls2.txt → dedup → final_paramurls.txt
  gf patterns → gf_xss.txt, gf_sqli.txt, gf_redirect.txt, gf_lfi.txt
```

## Legal

This tool is intended for authorized penetration testing and security research only. Only scan domains and systems you own or have explicit written permission to test.

## License

Add a license of your choice (e.g. MIT) before making this repository public.
