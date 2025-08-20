# 🪆 MATRYOSHKA v2.0
## Ephemeral Communication Analyzer & Nested Artifact Detection Tool

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform](https://img.shields.io/badge/platform-Windows%20%7C%20Linux%20%7C%20macOS-lightgrey)](https://github.com/your-username/matryoshka)

**MATRYOSHKA** is an advanced digital forensics tool designed to detect and analyze ephemeral communication channels and nested artifacts that malware uses to hide its tracks. Like a Russian nesting doll (Matryoshka), it progressively opens layers of hidden system artifacts to reveal covert communications.

## 🎯 Key Features

- **🔍 Multi-Layer Analysis**: Progressive examination of 6 distinct artifact layers
- **🚨 Threat Intelligence**: Automated IOC extraction with confidence scoring  
- **⚡ Enhanced Performance**: Concurrent processing for faster analysis
- **🔐 Privilege Awareness**: Adapts analysis depth based on execution privileges
- **📊 Comprehensive Reporting**: Detailed findings with threat level classification
- **🪟 Cross-Platform**: Full support for Windows, Linux, and macOS
- **📋 IOC Export**: Standard format export for SIEM and threat intelligence platforms
- **🧠 Memory Analysis**: Detection of shared memory and IPC artifacts
- **📝 Registry Scanning**: Windows registry analysis for persistence mechanisms
- **⏰ Timestomp Detection**: Identifies timestamp manipulation attempts

## 🏗️ Architecture

MATRYOSHKA employs a layered detection approach:

```
🪆 Layer 1: Surface Artifacts      - Recent temporary files & directories
🪆 Layer 2: Deletion Traces        - Cleared logs & emptied communication files  
🪆 Layer 3: Process Remnants       - Orphaned process artifacts & hidden directories
🪆 Layer 4: Network Shadows        - Covert channel traces & suspicious connections
🪆 Layer 5: Memory Artifacts       - Shared memory segments & IPC mechanisms
🪆 Layer 6: Registry Artifacts     - Windows persistence & configuration traces
```

## 🚀 Quick Start

### Prerequisites
```bash
pip install psutil argparse pathlib
```

### Basic Usage
```bash
# Standard analysis
python matryoshka.py --open

# Privileged analysis (recommended)
sudo python matryoshka.py --open --verbose

# Generate previous report
python matryoshka.py --report

# Export IOCs for threat intelligence
python matryoshka.py --export-iocs --output threats.json
```

### Custom Configuration
```bash
# Use custom detection rules
python matryoshka.py --config custom_config.json --open

# Specify custom database
python matryoshka.py --db investigation.db --open
```

## 📖 Configuration

MATRYOSHKA supports extensive customization through JSON configuration files:

```json
{
  "max_file_size": 104857600,
  "recent_threshold_hours": 2,
  "hash_algorithm": "sha256",
  "max_workers": 4,
  "suspicious_ports": [31337, 31338, 4444, 8080],
  "analysis_options": {
    "enable_registry_scan": true,
    "enable_memory_scan": true,
    "parallel_processing": true
  }
}
```

## 📊 Sample Output

```
🪆 MATRYOSHKA ENHANCED ANALYSIS REPORT 🪆
===============================================================================

🎯 THREAT SUMMARY:
   🔴 HIGH:     3 artifacts
   🟡 MEDIUM:   7 artifacts  
   🟢 LOW:     12 artifacts
   📊 TOTAL:   22 artifacts

🪆 LAYER 4: Network Shadows - 3 artifacts
   Threats: 🔴2 🟡1 🟢0
────────────────────────────────────────────────────────────
  🔴 [🔍🔍🔍🔍🔍🔍] covert_channel_trace
     📍 Location: 127.0.0.1:31337
     📝 Description: CRITICAL: Covert channel port 31337 - deep layer communication
     ⚠️  Threat Level: HIGH

🚨 INDICATORS OF COMPROMISE (IOCs):
   Total IOCs: 8
   High Confidence: 3

🚨 CRITICAL ALERT: Multiple high-threat artifacts detected!
   🔍 Immediate investigation recommended
   🛡️  System may be compromised
```

## 🛡️ Detection Capabilities

### Ephemeral Communication Channels
- **Temporary Files**: Recent artifacts in temp directories
- **Memory-Only Communications**: Shared memory segments and IPC
- **Network Covert Channels**: Suspicious port usage and connection patterns
- **Process Artifacts**: Orphaned process remnants and hidden directories

### Anti-Forensics Detection  
- **Timestamp Manipulation**: Detection of timestomping attempts
- **Log Clearing**: Recently emptied log files and communication traces
- **Hidden Artifacts**: Concealed files and registry entries
- **Privilege Escalation**: Protected artifact access attempts

### Malware Persistence
- **Registry Persistence**: Windows registry analysis for persistence mechanisms
- **Service Modifications**: System service tampering detection
- **Startup Programs**: Unauthorized startup entries
- **DLL Injection**: Suspicious library loading patterns

## 🔧 Advanced Usage

### Custom Analysis Workflows
```python
from matryoshka import Matryoshka

# Initialize with custom configuration
analyzer = Matryoshka(db_path="custom.db", config_path="rules.json")

# Run specific layer analysis
surface_findings = analyzer.peek_surface_layer()
network_findings = analyzer.discover_deepest_layer()

# Export findings
ioc_report = analyzer.export_ioc_report()
```

### Integration with SIEM Systems
```bash
# Export IOCs in standard format
python matryoshka.py --export-iocs --output iocs.json

# Integration example with Splunk/ELK
cat iocs.json | jq '.indicators.network_endpoints[] | select(.confidence > 0.8)'
```

## 🎛️ Command Line Options

| Option | Description |
|--------|-------------|
| `--open` | Execute full multi-layer analysis |
| `--report` | Display findings from previous sessions |
| `--export-iocs` | Export IOCs in JSON format |
| `--config FILE` | Use custom configuration file |
| `--db FILE` | Specify database file path |
| `--output FILE` | Output file for IOC export |
| `--verbose` | Enable detailed logging |
| `--monitor` | Real-time monitoring mode (future feature) |

## 🏢 Enterprise Features

### Threat Intelligence Integration
- **IOC Confidence Scoring**: Machine learning-based artifact classification
- **MITRE ATT&CK Mapping**: Technique identification and mapping
- **Threat Actor Profiling**: Behavioral pattern analysis
- **Timeline Construction**: Chronological artifact correlation

### Scalability & Performance
- **Concurrent Processing**: Multi-threaded analysis for large systems
- **Memory Optimization**: Efficient handling of large artifact datasets
- **Distributed Analysis**: Network-wide deployment capabilities
- **API Integration**: RESTful API for automated incident response

## 📈 Use Cases

### Digital Forensics
- **Incident Response**: Rapid triage of compromised systems
- **Malware Analysis**: Detection of covert communication mechanisms
- **Evidence Collection**: Comprehensive artifact documentation
- **Timeline Analysis**: Reconstruction of attack progression

### Threat Hunting
- **Proactive Detection**: Identification of advanced persistent threats
- **IOC Development**: Creation of custom detection signatures
- **Behavioral Analysis**: Pattern recognition in system artifacts
- **Intelligence Gathering**: Collection of threat intelligence data

### Security Operations
- **SIEM Enhancement**: Enrichment of security event data
- **Automated Analysis**: Integration with security orchestration platforms
- **Compliance Auditing**: Verification of security control effectiveness
- **Threat Assessment**: Risk evaluation of detected artifacts

## 🤝 Contributing

I wholeheartedly welcome all possible contributions! Please see the [Contributing Guidelines](CONTRIBUTING.md) for details.

### Development Setup
```bash
git clone https://github.com/your-username/matryoshka.git
cd matryoshka
pip install -r requirements.txt
python -m pytest tests/
```

## ⚠️ Legal Disclaimer

MATRYOSHKA is designed for legitimate digital forensics and cybersecurity purposes. Users are responsible for ensuring compliance with applicable laws and regulations. The authors assume no liability for misuse of this tool.

## 🙏 Acknowledgments

- Inspired by the concept of Russian nesting dolls (Matryoshka)
- Built upon the excellent `psutil` library for system analysis
- Thanks to the digital forensics community for inspiration and feedback!

---
