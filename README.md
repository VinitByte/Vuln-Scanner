VulnScanner — Automated Web Security Testing Dashboard
VulnScanner is a Flask-based web dashboard that orchestrates command-line tools for authorized web application security assessments. It provides a browser UI for starting scans, watching logs and progress, reviewing generated output files, and generating reports.
Authorization required: Use this project only against domains, hosts, and applications that you own or have explicit written permission to test. Scanning can create traffic, trigger alerts, affect availability, or violate a provider's terms. Do not scan public targets without permission. Start with a test lab or a deliberately vulnerable application.

Contents
- Features
- How it works
- Requirements
- 1. Download the project
- 2. Install system dependencies
- 3. Install Go-based tools
- 4. Install Python tools and helper repositories
- 5. Configure your PATH
- 6. Install Python dependencies
- 7. Start the dashboard
- 8. Run a scan
- 9. Results and reports
- Troubleshooting
- Project structure
- Limitations
- Disclaimer
Features
- Browser-based dashboard built with Flask, HTML, CSS, and JavaScript.
- Full pipeline for discovery and security checks.
- Separate vulnerability-only scan workflow.
- Live scan status and log polling, with a stop control.
- Output-file viewer and report generation, including PDF reports.
- Scan results are stored in timestamped folders under outputs/.
How it works
The Flask application (app.py) calls Bash scripts in scripts/. Each script invokes one or more external security tools. The web interface is served from templates/index.html.
The full pipeline currently runs these stages:
1. Subdomain enumeration — Subfinder
2. Live-host detection — httpx
3. Technology detection — WhatWeb
4. Port and service scan — Nmap
5. Subdomain takeover checks — Subjack
6. URL collection — GAU and Katana
7. URL filtering and parsing — uro and unfurl
8. JavaScript analysis — Katana, LinkFinder, SecretFinder, Gitleaks
9. Parameter discovery — ParamSpider and Arjun
10. Parameter URL construction/fuzzing — ffuf and GF patterns
11. Template-based checks — Nuclei
12. XSS/SQL injection checks — Dalfox and SQLMap
13. Server audit — Nikto
14. CORS checks — CORScanner
15. Web vulnerability checks — Wapiti
16. WAF detection — wafw00f
The vulnerability-only workflow runs Nuclei, Dalfox/SQLMap, Nikto, CORScanner, and wafw00f. Exact results depend on installed tools, tool versions, network access, target responses, and the scan configuration.
Requirements
This project is designed for Linux, particularly Kali Linux or a Debian/Ubuntu-based security-testing environment. The scripts use Bash and Linux utilities; they are not designed to run natively on Windows Command Prompt or PowerShell.
You will need:
- Python 3.10 or newer recommended
- pip and Python virtual-environment support
- Bash and standard Linux utilities (curl, git, grep, sed, awk, sort, tee, unzip, etc.)
- Go (for several ProjectDiscovery and other Go-based tools)
- The external tools listed below
- Internet access for installing tools and for scans
- Permission to access and test the target
Some tools may require separate configuration or additional dependencies. Not every external scanner is bundled in this repository.
1. Download the project
Clone your GitHub repository (replace the URL with your own repository URL):
git clone https://github.com/YOUR-USERNAME/YOUR-REPOSITORY.git
cd YOUR-REPOSITORY
If you downloaded the ZIP instead, extract it and open a terminal in the folder containing app.py, requirements.txt, and the scripts/ directory.
Important: In the ZIP provided for this README, the project files are inside the scanner_v25/ directory. If your GitHub repository has the same structure, run:
cd scanner_v25
Run ls and confirm that app.py, requirements.txt, templates/, and scripts/ are present in the current directory.
2. Install system dependencies
On Kali Linux / Debian / Ubuntu:
sudo apt update
sudo apt install -y python3 python3-pip python3-venv git curl wget unzip build-essential \
  nmap whatweb nikto wapiti sqlmap ffuf
If a package is unavailable in your distribution, install it using its official project documentation. Package names and availability can vary by OS version.
3. Install Go-based tools
Install Go using your distribution's package manager or the official Go installation instructions. Then verify it:
go version
Install the tools used by the scripts:
go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install github.com/projectdiscovery/httpx/cmd/httpx@latest
go install github.com/projectdiscovery/katana/cmd/katana@latest
go install github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
go install github.com/lc/gau/v2/cmd/gau@latest
go install github.com/hahwul/dalfox/v2@latest
go install github.com/tomnomnom/gf@latest
go install github.com/tomnomnom/unfurl@latest
go install github.com/haccer/subjack@latest
go install github.com/gitleaks/gitleaks/v8@latest
Check that Go's binary directory is in your PATH:
export PATH="$PATH:$(go env GOPATH)/bin"
To make this permanent for Bash:
echo 'export PATH="$PATH:$(go env GOPATH)/bin"' >> ~/.bashrc
source ~/.bashrc
Verify a few installations:
subfinder -h
httpx -h
katana -h
nuclei -h
gau --help
dalfox -h
gf -h
unfurl -h
gitleaks version
Note: Some tools can change their installation method, repository, or compatibility over time. If a go install command fails, consult that tool's official documentation rather than downloading binaries from untrusted sources.
4. Install Python tools and helper repositories
Create a tools directory:
mkdir -p ~/tools
Create the project's Python environment first (see Install Python dependencies). Activate it before installing Python packages:
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install uro arjun
LinkFinder
git clone https://github.com/GerbenJavado/LinkFinder.git ~/tools/LinkFinder
python -m pip install -r ~/tools/LinkFinder/requirements.txt
SecretFinder
git clone https://github.com/m4ll0k/SecretFinder.git ~/tools/SecretFinder
python -m pip install -r ~/tools/SecretFinder/requirements.txt
The JavaScript-analysis script may expect these repositories at the paths configured inside the script. If you install them somewhere else, update scripts/09_js_extraction.sh to match your paths.
GF pattern files
mkdir -p ~/.gf
git clone https://github.com/tomnomnom/gf.git ~/tools/gf
cp ~/tools/gf/examples/*.json ~/.gf/
git clone https://github.com/1ndianJ3lly/Gf-Patterns.git ~/tools/Gf-Patterns
cp ~/tools/Gf-Patterns/*.json ~/.gf/ 2>/dev/null || true
If the second pattern repository URL or layout has changed, use its current official repository instructions. Confirm that ~/.gf/ contains the pattern JSON files required by your installed GF setup.
Additional tools
- ParamSpider and Arjun are used for parameter discovery.
- CORScanner is used by the CORS-check stage and may need to be installed separately using its official instructions.
- wafw00f is used for WAF detection. Install it using your distribution package or its official Python package instructions.
- Nmap, WhatWeb, Nikto, Wapiti, SQLMap, and ffuf are system tools installed in the previous step.
For any tool not found by the scripts, install it and ensure its executable is available in PATH.
5. Configure your PATH
The app launches scripts from its own scripts/ directory, but those scripts call external tools by executable name or check common installation paths. Open a new terminal after changing your shell configuration.
Check tool availability:
command -v subfinder
command -v httpx
command -v katana
command -v nuclei
command -v gau
command -v dalfox
command -v gf
command -v unfurl
command -v nmap
command -v whatweb
command -v nikto
command -v wapiti
command -v sqlmap
command -v ffuf
command -v wafw00f
If any command prints nothing, that tool is either not installed or not on PATH. Fix that before expecting the corresponding scan stage to work.
6. Install Python dependencies
From the directory containing requirements.txt:
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
The included requirements file installs Flask and ReportLab. ReportLab is used for PDF report generation.
Whenever you open a new terminal, activate the environment again:
cd /path/to/YOUR-REPOSITORY
source .venv/bin/activate
If your project is inside scanner_v25/, cd into that folder before activating .venv and running the app.
7. Start the dashboard
Run from the project directory (the directory containing app.py):
source .venv/bin/activate
python app.py
When Flask starts, open this URL in your browser:
http://127.0.0.1:5000
If you are running the browser on the same computer, http://localhost:5000 also works.
Stop the local server with Ctrl+C in the terminal.
Accessing from another device
By default, use the app locally. Do not expose this scanner dashboard directly to the public internet. It does not provide a production-grade authentication layer. If remote access is necessary, use a trusted private network and suitable access controls.
8. Run a scan
1. Start the Flask app.
2. Open http://127.0.0.1:5000.
3. Enter a domain or URL for a target you are authorized to test.
4. Select the full scan or vulnerability-only scan option available in the UI.
5. Start the scan and monitor the progress/log panel.
6. Use the stop control if you need to interrupt a scan.
7. When the scan finishes, open the Outputs/results area to inspect the files.
Use a test target first. Some stages perform active requests or fuzzing and may generate significant traffic. Start with conservative testing and only enable checks that are allowed by the target owner.
9. Results and reports
Scan output is stored under the project's outputs/ directory, usually in a folder named after the target and a timestamp:
outputs/
└── example.com_YYYYMMDD_HHMMSS/
    ├── subdomains.txt
    ├── live.txt
    ├── whatweb.txt
    ├── nmap.txt
    ├── allurls.txt
    ├── filtered_urls.txt
    ├── nuclei.txt
    └── ...
The actual files depend on which stages ran and whether the tools returned results. Use the dashboard's Outputs area to browse results and the report controls to generate HTML/text or PDF reports if available in your version.
A tool producing an empty output file does not prove that the target is secure. It can mean no findings were detected, the tool was missing, the request failed, the target blocked scanning, or the stage did not complete successfully.
Troubleshooting
ModuleNotFoundError: No module named 'flask'
Activate the virtual environment and install requirements:
source .venv/bin/activate
python -m pip install -r requirements.txt
ModuleNotFoundError: No module named 'reportlab'
source .venv/bin/activate
python -m pip install reportlab
bash: ...: command not found or a stage says a tool is missing
Install the missing tool using its official instructions and ensure its binary directory is on PATH. Re-run the command -v TOOL_NAME checks above.
A tool works in the terminal but not in the app
The Flask process may have been started before PATH was updated, or the tool may be installed in a different user's home directory. Update PATH, restart the terminal, reactivate .venv, and restart Flask.
Permission denied for a script
From the project root:
chmod +x scripts/*.sh
The app calls the scripts using bash, so executable permission is not always required, but this command can help when running scripts manually.
Port 5000 is already in use
Stop the other process using port 5000, or change the Flask port in the application's startup code. Make sure to open the matching URL in your browser.
Scan finishes quickly or outputs are empty
- Check the live logs for missing tools and error messages.
- Confirm the target is entered correctly and resolves from your machine.
- Confirm the target is authorized and reachable.
- Check that all required external tools are installed and configured.
- Remember that some stages depend on files created by earlier stages.
Python package installation fails on Kali/Debian
Some distributions restrict global pip installations. Use the virtual environment described above instead of forcing a system-wide installation.
Project structure
YOUR-REPOSITORY/
├── app.py                    # Flask application and scan orchestration
├── generate_pdf_report.py    # PDF report generation
├── requirements.txt          # Python dependencies
├── README.md
├── templates/
│   └── index.html             # Web dashboard
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
└── outputs/                   # Created automatically for scan results
The outputs/ directory is generated when the app starts and should generally not be committed if it contains target data or sensitive scan findings.
Limitations
- This project coordinates external tools; it is not a replacement for manual security review or a professional penetration test.
- Tool availability, versions, command-line options, and output formats can change.
- Some stages may be skipped or return incomplete results if a dependency is missing or a target blocks requests.
- Findings can include false positives and must be validated safely.
- The application should be treated as a local development/testing tool, not a hardened multi-user production service.
Disclaimer
This repository is provided for educational and authorized security-testing purposes only. You are responsible for obtaining permission, complying with applicable laws and policies, and ensuring your tests do not disrupt systems or expose data. The authors are not responsible for misuse or damage caused by running this software.
