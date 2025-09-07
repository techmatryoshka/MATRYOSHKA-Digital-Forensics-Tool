# 🎎 MATRYOSHKA v2.0
## Ephemeral Communication Analyzer & Nested Artifact Detection Tool

[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Security: Enhanced](https://img.shields.io/badge/Security-Enhanced-green.svg)](https://github.com/your-username/matryoshka)

A secure, enterprise-grade digital forensics tool for uncovering hidden layers of ephemeral communication channels and nested artifacts. Like a Russian nesting doll, Matryoshka reveals what's hidden deep within systems through systematic layer analysis.

## Features

### Core Capabilities
- **Multi-layer artifact detection** across filesystem, memory, network, and registry
- **Advanced timestomp detection** with multiple behavioral indicators
- **Threat scoring system** with formal confidence metrics
- **Cross-platform support** for Windows, Linux, and macOS
- **Chain of custody tracking** for forensic evidence integrity
- **IOC extraction and export** in standard formats

### Security Enhancements
- **Input validation and sanitization** with path traversal protection
- **Secure database operations** with integrity checks
- **Memory-safe file operations** with configurable size limits
- **Privilege escalation detection** and secure temporary file handling
- **Configuration parameter validation** with type checking
- **Comprehensive audit logging** with rotation support

### Forensic Methodology
- **Evidence-based analysis** with unique identifiers and timestamps
- **Formal threat assessment** using weighted indicator scoring
- **Session integrity** with cryptographic verification
- **Detailed reporting** with provenance tracking
- **Concurrent processing** for performance optimization

## Installation

### Prerequisites
- Python 3.8 or higher
- Required system privileges for deep analysis (administrator/root recommended)

**Required packages:**
- `psutil >= 5.8.0`
- `sqlite3` (included with Python)
- `pathlib` (included with Python 3.4+)

### Quick Install
```bash
git clone https://github.com/techmatryoshka/MATRYOSHKA-Digital-Forensics-Tool.git
cd matryoshka
pip install -r requirements.txt
```

## Usage

### Basic Analysis
```bash
# Standard analysis with default settings
python matryoshka.py --analyze

# Analysis with elevated privileges (recommended)
sudo python matryoshka.py --analyze

# Analysis with detailed logging
python matryoshka.py --analyze --log-level DEBUG --log-file analysis.log
```

### Advanced Usage
```bash
# Custom configuration with report export
python matryoshka.py --analyze --config custom_config.json --export-report

# Export existing findings to report
python matryoshka.py --export-report --output detailed_report.json

# Specify custom database and working directory
python matryoshka.py --analyze --db /secure/location/evidence.db --work-dir /tmp/analysis
```

### Configuration File
Create a JSON configuration file for custom settings:

```json
{
  "max_file_size": 104857600,
  "recent_threshold_hours": 2,
  "hash_algorithm": "sha256",
  "max_workers": 4,
  "max_depth_analysis": 1000,
  "suspicious_ports": [4444, 5555, 6666, 31337, 31338, 31339],
  "temp_paths_unix": ["/tmp", "/var/tmp", "/dev/shm"],
  "temp_paths_windows": ["%TEMP%", "%TMP%", "C:\\Windows\\Temp"]
}
```

## Command Line Options

| Option | Description |
|--------|-------------|
| `--analyze` | Run comprehensive forensic analysis |
| `--export-report` | Export findings in JSON report format |
| `--config FILE` | Use custom configuration file (JSON) |
| `--db FILE` | Custom database file path |
| `--output FILE` | Output file for reports |
| `--log-file FILE` | Log file path |
| `--log-level LEVEL` | Logging level (DEBUG/INFO/WARNING/ERROR) |
| `--work-dir DIR` | Working directory for temporary files |

## Analysis Layers

Matryoshka analyzes systems through multiple detection layers:

### Layer 1: Surface Artifacts
- Recent temporary files and cache entries
- Hidden files and suspicious naming patterns
- File access time analysis with security validation

### Layer 2: Memory Artifacts
- Shared memory segments (/dev/shm analysis)
- Memory-mapped files and IPC traces
- Semaphore and communication artifacts

### Layer 3: Process Remnants
- Orphaned process artifacts and traces
- Hidden directories and communication endpoints
- Process communication shadows

### Layer 4: Network Shadows
- Suspicious network connections and ports
- Covert channel detection on non-standard ports
- Local communication loop analysis

### Layer 5: Registry Artifacts (Windows)
- Ephemeral registry keys and values
- Persistence mechanism detection
- Recently modified registry entries

## Threat Assessment

### Threat Levels
- **CRITICAL**: Multiple high-confidence indicators, potential system compromise
- **HIGH**: Advanced techniques detected, immediate investigation recommended
- **MEDIUM**: Suspicious activities requiring monitoring
- **LOW**: Standard system artifacts with minimal risk

### Confidence Scoring
Confidence levels (0.0-1.0) based on:
- Indicator reliability and correlation
- Evidence quality and completeness
- Analysis depth and verification
- Historical threat intelligence

## Security Features

### Data Protection
- Database files created with restrictive permissions (0o600)
- Secure temporary file handling with automatic cleanup
- Memory-safe operations with configurable limits
- Input validation preventing directory traversal attacks

### Audit Trail
- Complete chain of custody for all evidence
- Session integrity with cryptographic verification
- Comprehensive logging with rotation support
- Evidence provenance tracking

### Access Control
- Privilege level detection and reporting
- Secure database operations with foreign key constraints
- Configuration parameter validation
- Resource usage limits and monitoring

## Output Formats

### Analysis Report Structure
```json
{
  "metadata": {
    "tool": "Matryoshka v2.0",
    "export_timestamp": "2025-01-15T10:30:00",
    "session_id": "SESSION_1642246200_1234",
    "total_findings": 42
  },
  "findings": [
    {
      "evidence_id": "EVID_1642246200123456_1234",
      "timestamp": "2025-01-15T10:25:00",
      "artifact_type": "surface_artifact",
      "location": "/tmp/suspicious_file.tmp",
      "threat_level": "HIGH",
      "confidence": 0.85,
      "indicators": ["hidden_files", "suspicious_naming"],
      "chain_of_custody": [...]
    }
  ]
}
```

## Performance Considerations

- **Concurrent Processing**: Configurable worker threads for directory analysis
- **Memory Management**: File size limits prevent memory exhaustion
- **Database Optimization**: Indexed operations with WAL mode
- **Resource Limits**: Configurable analysis depth to prevent infinite recursion

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/enhancement`)
3. Commit your changes (`git commit -am 'Add new detection capability'`)
4. Push to the branch (`git push origin feature/enhancement`)
5. Create a Pull Request

### Development Guidelines
- Follow PEP 8 style guidelines
- Add type hints for new functions
- Include comprehensive docstrings
- Write unit tests for new features
- Update documentation for API changes

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Security Considerations

### Responsible Use
This tool is designed for legitimate forensic investigation and system administration purposes. Users are responsible for:
- Obtaining proper authorization before analysis
- Complying with applicable laws and regulations
- Protecting sensitive information discovered during analysis
- Following organizational security policies

### Limitations
- Requires appropriate system privileges for complete analysis
- Some artifacts may be missed on heavily restricted systems
- Anti-forensics techniques may limit detection capabilities
- Performance varies based on system resources and permissions

## CHANGELOG

### v2.0 (Current)
- Enhanced security validation with path traversal protection
- Formal threat scoring system with confidence metrics
- Chain of custody tracking for evidence integrity
- Advanced timestomp detection algorithms
- Secure database operations with integrity checks
- Comprehensive logging with rotation support
- Memory-safe file operations with size limits

### v1.0
- Initial multi-layer analysis implementation
- Cross-platform compatibility
- Basic threat detection and IOC export
- Database storage for findings
- Concurrent processing support

## Support

- **Issues**: Please report bugs and feature requests via GitHub Issues
- **Documentation**: Additional documentation will be uploaded within the repository if needed
- **Security**: Report security vulnerabilities privately to matlabmatryoshka@gmail.com
