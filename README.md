# MATRYOSHKA - An open-source ethical hacking tool (COMMING SOON)
A digital forensics tool that detects and analyzes ephemeral communication channels and temporary artifacts left behind by malware, deleted files, and covert operations.

## Features

- **Temporary Artifact Hunter**: Scans temp directories for recently accessed files
- **Deletion Signature Detector**: Finds evidence of recently deleted files
- **Process Ghost Scanner**: Detects remnants of terminated processes
- **Network Echo Analyzer**: Identifies traces of closed network connections
- **SQLite Evidence Database**: Persistent storage of all findings
- **Confidence Scoring**: Rates the reliability of each detection
- **Timeline Reconstruction**: Builds chronological view of ghost activities

## Quick Start

### Installation
```bash
git clone https://github.com/matlabmatryoshka/MATRYOSHKA
cd MATRYOSHKA
chmod +x install.sh
sudo ./install.sh
```

### Usage
```bash
# Run comprehensive scan
MATRYOSHKA --scan

# View previous findings
MATRYOSHKA --report

# Use short alias
MATR --scan
```

## Sample Output
```
[+] MATRYOSHKA v1.0 - Starting comprehensive scan...
[*] Scanning temporary directories for ghost artifacts...
[*] Scanning for deleted file signatures...
[*] Scanning for hidden process artifacts...
[*] Scanning for network connection echoes...

============================================================
MATRYOSHKA DETECTION REPORT
============================================================

[TEMPORARY_ARTIFACTS] - 3 artifacts found:
----------------------------------------
  [●●●●●●●○○○] recent_temp_file
     Location: /tmp/.hidden_comm_12345
     Description: Recently accessed temp file (accessed 45s ago)
     Timestamp: 2025-08-17T14:30:25

[NETWORK_ECHOES] - 1 artifacts found:
----------------------------------------
  [●●●●○○○○○○] recent_connection_close
     Location: 192.168.1.100:8080
     Description: Recently closed connection on unusual port 8080
     Timestamp: 2025-08-17T14:29:15

Total hidden artifacts detected: 4
```

## Advanced Features

### Database Analysis
```bash
# View database directly
sqlite3 matryoshka.db
sqlite> SELECT * FROM natryoshka_findings ORDER BY confidence_score DESC;
```

### Custom Scans
```python
from MATRYOSHKA import MATRYOSHKA

matryoshka = MATRYOSHKA()
matryoshka.scan_temp_directories()
matryoshka.generate_report()
```

## Detection Categories

- **temporary_artifacts**: Recently accessed temp files
- **deletion_artifacts**: Evidence of deleted files
- **process_matryoshka**: Remnants of terminated processes  
- **network_echoes**: Traces of closed connections

## Requirements

- Python 3.6+
- psutil library
- Linux/Windows/macOS support
- Root/Administrator privileges recommended

## Contributing

1. Fork the repository
2. Create a feature branch
3. Add new detection modules
4. Submit pull request

## Roadmap

- [ ] Memory dump analysis integration
- [ ] Registry ghost detection (Windows)
- [ ] Network packet reconstruction
- [ ] Machine learning anomaly detection
- [ ] Web dashboard interface
- [ ] Integration with YARA rules
