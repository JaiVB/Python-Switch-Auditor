# Switch Configuration Auditor

[![Status](https://img.shields.io/badge/status-in--progress-orange.svg)](https://github.com)
[![Python](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> 🚧 **Work in Progress:** This project is actively under initial development. Core parsing components, security rule definitions, and the CLI interface are currently being implemented and tested.

A lightweight Python CLI utility designed to audit Cisco IOS switch configurations against baseline security practices and administrative standards. 

Rather than relying on flat regular expressions across entire configuration dumps, this tool parses hierarchical configuration blocks (interfaces, VTY lines, and global commands) to evaluate contextual compliance, detect insecure services, and identify configuration drift.

---

## Roadmap & Current Progress

- [x] Repository architecture & directory scaffolding
- [x] Project documentation & sample switch fixtures
- [ ] Hierarchical configuration parser (`auditor/parser.py`)
- [ ] Security rules engine (`auditor/rules.py`)
  - [ ] Cleartext remote management audit (`transport input telnet`)
  - [ ] Port security enforcement on operational access ports
  - [ ] Administrative description hygiene on active interfaces
- [ ] Terminal reporting module (`auditor/reporter.py`)
- [ ] CLI argument interface & exit codes (`main.py`)
- [ ] Automated unit test suite (`tests/test_rules.py`)

---

## Key Features (Target Implementation)

- **Hierarchical Block Parsing:** Groups nested configuration commands under their respective interface (`interface GigabitEthernet0/1`) and line configuration contexts (`line vty 0 4`).
- **Security & Hygiene Rules:**
  - **Insecure Transport Detection:** Flags cleartext management protocols (`transport input telnet` or unhardened `transport input all`) as critical risks.
  - **Port Security Compliance:** Verifies that operational access switchports enforce Layer 2 port security controls (`switchport port-security`).
  - **Interface Documentation Baselines:** Identifies active (non-shutdown) interfaces lacking an administrative `description`.
- **Severity-Based Reporting:** Categorizes audit findings into `CRITICAL`, `WARNING`, and `INFO` levels with actionable remediation recommendations.
- **Automation Ready:** Emits structured exit codes suitable for CI/CD pipeline gating or network pre-commit checks.

---

## Repository Structure

```text
switch-config-auditor/
│
├── auditor/
│   ├── __init__.py
│   ├── parser.py        # Logic for tokenizing and grouping hierarchical IOS blocks
│   ├── rules.py         # Modular audit rules and condition checks
│   └── reporter.py      # Terminal formatting and output rendering
│
├── configs/             # Test fixtures and baseline switch configurations
│   ├── sample_compliant.cfg
│   └── sample_vulnerable.cfg
│
├── tests/               # Automated unit tests for rule validation
│   ├── __init__.py
│   └── test_rules.py
│
├── main.py              # CLI entry point (argument parsing & workflow orchestration)
├── requirements.txt     # Optional styling and development dependencies
├── .gitignore
└── README.md
```

---

## Getting Started

### Prerequisites
- Python 3.8+
- Zero external dependencies required (core engine utilizes the Python Standard Library).

### Installation

Clone the repository:
```bash
git clone https://github.com/<YOUR-GITHUB-USERNAME>/switch-config-auditor.git
cd switch-config-auditor
```

*(Optional)* Create and activate a virtual environment:
```bash
python3 -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

---

## Usage

Run an audit against any saved running-config or startup-config text file:

```bash
# Basic scan
python3 main.py --config configs/sample_vulnerable.cfg

# Filter findings by minimum severity
python3 main.py --config configs/sample_vulnerable.cfg --min-severity WARNING
```

### Sample Expected Output

```text
============================================================
              SWITCH CONFIGURATION AUDIT REPORT             
============================================================
Target File: configs/sample_vulnerable.cfg
Hostname:    SW-ACCESS-01

[CRITICAL] Insecure Remote Management Allowed
  Block:       line vty 0 4
  Issue:       Telnet is permitted for management access ('transport input telnet').
  Remediation: Enforce SSH access only:
               SW-ACCESS-01(config-line)# transport input ssh

[WARNING] Missing Port Security on Access Port
  Block:       interface GigabitEthernet0/1
  Issue:       Operational access port lacks 'switchport port-security'.
  Remediation: Configure port security controls:
               SW-ACCESS-01(config-if)# switchport port-security
               SW-ACCESS-01(config-if)# switchport port-security violation restrict

[INFO] Missing Interface Description
  Block:       interface GigabitEthernet0/1
  Issue:       Active interface does not have an administrative description defined.
  Remediation: Add a descriptive label identifying the connected endpoint:
               SW-ACCESS-01(config-if)# description <PURPOSE_OR_DEVICE>

------------------------------------------------------------
Audit Complete: 1 CRITICAL, 1 WARNING, 1 INFO finding(s).
============================================================
```

---

## Running Tests

Execute the unit test suite to verify rule detection logic:

```bash
python3 -m unittest discover tests
```

---

## License

This project is licensed under the MIT License.