# 🔍 VulnScanner — Automated Web Security Testing Dashboard

VulnScanner is a Flask-based web application that provides a browser-based dashboard for automated web security testing. It integrates multiple command-line security tools into a single workflow, helping users perform reconnaissance, discover endpoints, identify potential vulnerabilities, monitor scan progress, and generate reports.

The project is designed for educational purposes, authorized penetration testing, and security research.

> ⚠️ **Disclaimer:** Use this tool only on websites, domains, and applications that you own or have explicit permission to test. Unauthorized scanning may violate laws, policies, or service terms.

---

## 📌 Features

- **Web-Based Dashboard:** Simple interface built using Flask, HTML, CSS, and JavaScript.
- **Automated Reconnaissance:** Discovers subdomains, live hosts, technologies, and URLs.
- **Port Scanning:** Identifies open ports and services using Nmap.
- **JavaScript Analysis:** Extracts endpoints and checks for potentially exposed secrets.
- **Parameter Discovery:** Finds URL parameters for further security testing.
- **Vulnerability Scanning:** Integrates Nuclei, Dalfox, SQLMap, Nikto, and other tools.
- **WAF Detection:** Detects Web Application Firewalls using wafw00f.
- **Live Scan Monitoring:** Displays scan progress and logs through the dashboard.
- **Scan Management:** Supports stopping running scans through the interface.
- **Report Generation:** Generates reports, including PDF reports.
- **Organized Results:** Stores scan outputs in target-specific directories.

---

## 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| Python | Application logic |
| Flask | Web framework |
| HTML, CSS, JavaScript | Dashboard interface |
| Bash | Automation and scan orchestration |
| Go | Installation and execution of several security tools |
| ReportLab | PDF report generation |
| Nmap | Port and service scanning |
| Subfinder | Subdomain discovery |
| httpx | Live-host detection |
| Katana | Web crawling |
| Nuclei | Template-based vulnerability detection |
| SQLMap | SQL injection testing |
| Dalfox | XSS testing |
| Nikto | Web server security checks |
| Wapiti | Web application vulnerability scanning |
| WhatWeb | Technology fingerprinting |

---

## ⚙️ System Requirements

This project is primarily designed for Linux environments, such as Kali Linux, Ubuntu, and Debian.

Required software:

- Python 3.10 or newer recommended
- pip
- Python virtual environment support
- Git
- Bash
- Go
- curl and wget
- Nmap and other required security tools
- An active internet connection for installation and scanning

**Note:** The project uses Linux shell scripts and is not designed to run directly in Windows Command Prompt or PowerShell. Windows users can use WSL2 with a supported Linux distribution.

---

# 🚀 Installation and Setup

Follow these steps to install and run the project.

## Step 1: Clone the Repository

Open your Linux terminal and execute:

```bash
git clone https://github.com/VinitByte/Vuln-Scanner.git
```

Navigate to the downloaded project directory:

```bash
cd YOUR-REPOSITORY
```

Replace `YOUR-USERNAME` and `YOUR-REPOSITORY` with your actual GitHub username and repository name.

If the project files are inside a subdirectory such as `scanner_v25`, navigate into that directory:

```bash
cd scanner_v25
```

Confirm that the directory contains `app.py`, `requirements.txt`, `templates/`, and `scripts/`.

```bash
ls
```

## Step 2: Install System Dependencies

Update your package lists:

```bash
sudo apt update
```

Install the commonly required system packages:

```bash
sudo apt install -y python3 python3-pip python3-venv git curl wget unzip build-essential nmap whatweb nikto wapiti sqlmap ffuf
```

Some tools may need to be installed separately if they are unavailable in your distribution's repositories.

## Step 3: Install Go

Install Go using your Linux distribution's package manager:

```bash
sudo apt install -y golang-go
```

Verify the installation:

```bash
go version
```

If your distribution's Go version is too old for a required tool, install a supported version using the official Go installation instructions.

## Step 4: Install Go-Based Security Tools

Install the tools used by the scanning scripts:

```bash
go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
```

```bash
go install github.com/projectdiscovery/httpx/cmd/httpx@latest
```

```bash
go install github.com/projectdiscovery/katana/cmd/katana@latest
```

```bash
go install github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
```

```bash
go install github.com/lc/gau/v2/cmd/gau@latest
```

```bash
go install github.com/hahwul/dalfox/v2@latest
```

```bash
go install github.com/tomnomnom/gf@latest
```

```bash
go install github.com/tomnomnom/unfurl@latest
```

```bash
go install github.com/haccer/subjack@latest
```

```bash
go install github.com/gitleaks/gitleaks/v8@latest
```

Some installation commands may change as upstream projects evolve. If a command fails, consult the tool's official documentation.

### Configure the PATH

Add Go's binary directory to your PATH:

```bash
echo 'export PATH="$PATH:$(go env GOPATH)/bin"' >> ~/.bashrc
```

Reload the shell configuration:

```bash
source ~/.bashrc
```

Verify that the commands are available:

```bash
subfinder -h
httpx -h
katana -h
nuclei -h
gau --help
dalfox -h
gf -h
unfurl -h
```

## Step 5: Install Python Helper Tools

Create a directory for helper tools:

```bash
mkdir -p ~/tools
```

Install the Python helper packages in your project's virtual environment after completing Step 6:

```bash
python -m pip install uro arjun
```

### Install LinkFinder

```bash
git clone https://github.com/GerbenJavado/LinkFinder.git ~/tools/LinkFinder
```

```bash
python -m pip install -r ~/tools/LinkFinder/requirements.txt
```

### Install SecretFinder

```bash
git clone https://github.com/m4ll0k/SecretFinder.git ~/tools/SecretFinder
```

```bash
python -m pip install -r ~/tools/SecretFinder/requirements.txt
```

The JavaScript extraction script may expect helper repositories at particular paths. Check `scripts/09_js_extraction.sh` and update those paths if necessary.

### Configure GF Patterns

```bash
mkdir -p ~/.gf
git clone https://github.com/tomnomnom/gf.git ~/tools/gf
cp ~/tools/gf/examples/*.json ~/.gf/
```

Additional GF patterns can be installed from a trusted pattern repository.

### Other Dependencies

Depending on the scan stages you use, you may also need:

- ParamSpider
- CORScanner
- wafw00f
- Nmap
- WhatWeb
- Nikto
- Wapiti
- SQLMap
- ffuf

Install these tools using their official installation instructions and ensure their executable files are available in your PATH.

---

## Step 6: Create a Python Virtual Environment

From the directory containing `app.py` and `requirements.txt`, execute:

```bash
python3 -m venv .venv
```

Activate the environment:

```bash
source .venv/bin/activate
```

Upgrade pip:

```bash
python -m pip install --upgrade pip
```

Install the project dependencies:

```bash
python -m pip install -r requirements.txt
```

The project's Python requirements include Flask and ReportLab for the web application and PDF report generation.

Whenever you open a new terminal, activate the environment again before running the application.

---

# ▶️ Running the Application

## Step 7: Start the Flask Server

Make sure you are in the project directory and the virtual environment is activated.

Run:

```bash
python app.py
```

Once the Flask server starts, open your browser and visit:

**http://127.0.0.1:5000**

You can also use:

**http://localhost:5000**

The VulnScanner dashboard should now be available in your browser.

To stop the Flask server, press `Ctrl + C` in the terminal.

> **Security note:** Keep the dashboard local unless you have configured appropriate authentication and network access controls. Do not expose an unauthenticated scanning interface directly to the public internet.

---

# 🔎 How to Use VulnScanner

## Step 8: Run a Security Scan

1. Start the Flask application.
2. Open `http://127.0.0.1:5000` in your browser.
3. Enter the domain or URL you are authorized to test.
4. Select the available full-scan or vulnerability-only workflow.
5. Start the scan.
6. Monitor the progress and logs displayed in the dashboard.
7. Use the stop control if you need to interrupt the scan.
8. After completion, review the generated output files and reports.

Start with a controlled test environment before scanning a production application.

Some scanning tools actively send requests, crawl pages, or fuzz parameters. Use only the testing intensity authorized by the system owner.

---

## 🔬 Scan Pipeline

The full scan workflow integrates several security-testing stages.

| Stage | Tool | Purpose |
|---|---|---|
| 1 | Subfinder | Subdomain enumeration |
| 2 | httpx | Live-host detection |
| 3 | WhatWeb | Technology identification |
| 4 | Nmap | Port and service scanning |
| 5 | Subjack | Subdomain takeover checks |
| 6 | GAU and Katana | URL collection and crawling |
| 7 | uro and unfurl | URL filtering and parsing |
| 8 | LinkFinder, SecretFinder, Gitleaks | JavaScript endpoint and secret analysis |
| 9 | ParamSpider and Arjun | Parameter discovery |
| 10 | ffuf and GF | Parameter fuzzing and pattern-based filtering |
| 11 | Nuclei | Template-based vulnerability checks |
| 12 | Dalfox and SQLMap | XSS and SQL injection testing |
| 13 | Nikto | Web server security checks |
| 14 | CORScanner | CORS configuration checks |
| 15 | Wapiti | Web application vulnerability scanning |
| 16 | wafw00f | WAF detection |

The vulnerability-only workflow focuses on vulnerability checks and selected server-security and WAF checks.

The exact results depend on installed tool versions, target behavior, network connectivity, configuration, and whether each stage completes successfully.

---

# 📁 Output Files and Reports

## Step 9: Find the Scan Results

Scan outputs are stored under the project's `outputs/` directory.

A typical output directory may look like this:

```text
outputs/
└── target_YYYYMMDD_HHMMSS/
    ├── subdomains.txt
    ├── live.txt
    ├── whatweb.txt
    ├── nmap.txt
    ├── allurls.txt
    ├── filtered_urls.txt
    ├── nuclei.txt
    └── other scan results
```

The exact filenames depend on the scan stages that ran.

Use the dashboard's output viewer to inspect generated files. Use the report controls provided by your version of the application to generate reports, including PDF reports where supported.

### Important Interpretation Note

An empty output file does not prove that a website is secure. It may indicate that no findings were detected, the target blocked requests, a tool was missing, or a scan stage failed.

Review scan logs and manually validate important findings before drawing conclusions.

---

# 🧰 Troubleshooting

## 1. Flask Module Not Found

If you see:

```text
ModuleNotFoundError: No module named 'flask'
```

Activate the virtual environment and install the dependencies:

```bash
source .venv/bin/activate
python -m pip install -r requirements.txt
```

## 2. ReportLab Module Not Found

If PDF generation fails because ReportLab is missing:

```bash
source .venv/bin/activate
python -m pip install reportlab
```

## 3. Command Not Found

If a scan stage reports that a command is missing, verify the executable:

```bash
command -v subfinder
command -v httpx
command -v katana
command -v nuclei
command -v nmap
command -v sqlmap
```

Install any missing tools and ensure their binary directories are included in your PATH.

## 4. Permission Denied

If you receive a permission error when running shell scripts:

```bash
chmod +x scripts/*.sh
```

Then restart the application and try again.

## 5. Port 5000 Already in Use

If Flask cannot start because port 5000 is occupied, stop the process using that port or configure the application to use another port.

Open the corresponding address in your browser after changing the port.

## 6. Scan Completes Without Results

Check the following:

- The target domain or URL is correct.
- The target is reachable from your machine.
- Required security tools are installed.
- Tool paths are correctly configured.
- The scan logs do not contain dependency or network errors.
- Earlier scan stages completed and generated the files needed by later stages.

## 7. Python Installation Errors on Kali Linux

Some Linux distributions restrict global Python package installations. Use the virtual environment described in this README instead of forcing a system-wide pip installation.

---

# 📂 Project Structure

```text
YOUR-REPOSITORY/
│
├── app.py
├── generate_pdf_report.py
├── requirements.txt
├── README.md
│
├── templates/
│   └── index.html
│
├── scripts/
│   ├── 01_subfinder.sh
│   ├── 02_httpx.sh
│   ├── 03_whatweb.sh
│   ├── 04_nmap.sh
│   ├── 05_subjack.sh
│   ├── 06_url_collection.sh
│   ├── 07_url_filter.sh
│   ├── 09_js_extraction.sh
│   ├── 10_param_discovery.sh
│   ├── 11_paramurls.sh
│   ├── 12_nuclei.sh
│   ├── 13_dalfox_sqlmap.sh
│   ├── 14_nikto.sh
│   ├── 15_cors.sh
│   ├── 16_wapiti.sh
│   └── 19_waf.sh
│
└── outputs/
```

The `outputs/` directory is created by the application when needed. Actual files and directory contents may vary with your repository version.

---

# 🔐 Security and Responsible Use

VulnScanner is intended for educational use, authorized penetration testing, and defensive security research.

- Obtain permission before scanning any system.
- Follow the target owner's scope and testing restrictions.
- Avoid disruptive tests against production systems.
- Protect scan reports because they may contain sensitive information.
- Do not commit credentials, API keys, tokens, or private scan results to GitHub.
- Validate findings before reporting them as confirmed vulnerabilities.

---

# ⚠️ Limitations

- The application depends on external command-line tools.
- Tool versions and command-line options may change.
- Scanning results can contain false positives or false negatives.
- Some tools may require additional configuration or API access.
- A missing dependency may cause individual pipeline stages to fail.
- The dashboard is intended as a local development and testing application, not a production-hardened multi-user service.

---

# 📜 Disclaimer

This project is provided for educational and authorized security-testing purposes only. Users are responsible for ensuring that their activities comply with applicable laws, regulations, service terms, and authorization requirements.

The developers assume no responsibility for misuse, unauthorized scanning, service disruption, or damage resulting from the use of this software.

---

**If you find this project useful, consider giving the repository a ⭐ on GitHub.**
