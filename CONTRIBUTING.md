# Contributing to MATRYOSHKA

Thank you for your interest in contributing to MATRYOSHKA! This project thrives on community contributions from digital forensics professionals, security researchers, and developers passionate about cybersecurity tools.

## 🎯 Project Vision

MATRYOSHKA aims to be the premier open-source tool for detecting ephemeral communication channels and nested artifacts used by advanced malware and APT groups. We welcome contributions that advance this mission while maintaining the highest standards of code quality and forensic integrity.

## 🚀 Ways to Contribute

### 🔍 **Forensic Research & Detection**
- **New Detection Layers**: Implement additional analysis layers (Layer 7+)
- **Platform-Specific Modules**: Windows, macOS, Linux-specific artifact detection
- **Malware Family Signatures**: Detection patterns for specific malware families
- **Anti-Forensics Detection**: New techniques for detecting evasion methods
- **IOC Enhancement**: Improved indicator extraction and classification

### 💻 **Code Contributions**
- **Performance Optimizations**: Faster scanning algorithms and concurrent processing
- **New Features**: Real-time monitoring, web interface, API endpoints
- **Bug Fixes**: Resolve issues reported in GitHub Issues
- **Code Quality**: Refactoring, documentation, and test coverage improvements
- **Cross-Platform Support**: Enhanced compatibility across operating systems

### 📚 **Documentation & Education**
- **Technical Documentation**: API docs, architecture guides, forensic methodologies
- **Usage Examples**: Real-world case studies and investigation workflows
- **Educational Content**: Tutorials, webinars, conference presentations
- **Translation**: Multi-language support for global forensics community

### 🧪 **Testing & Validation**
- **Malware Testing**: Controlled testing with malware samples (in isolated environments)
- **False Positive Analysis**: Reducing false positives while maintaining detection accuracy
- **Performance Benchmarking**: Testing on various system configurations
- **Regression Testing**: Ensuring new features don't break existing functionality

## 🛠️ Development Setup

### Prerequisites
```bash
# System requirements
- Python 3.8+
- Git
- SQLite 3
- Administrative/root privileges for full testing

# Recommended: Virtual environment for isolated testing
- VMware/VirtualBox for malware testing
- Kali Linux or similar forensics distribution
```

### Environment Setup
```bash
# Clone the repository
git clone https://github.com/your-username/matryoshka.git
cd matryoshka

# Create virtual environment
python3 -m venv matryoshka-env
source matryoshka-env/bin/activate  # Linux/macOS
# or
matryoshka-env\Scripts\activate     # Windows

# Install dependencies
pip install -r requirements.txt
pip install -r requirements-dev.txt  # Development dependencies

# Install pre-commit hooks
pre-commit install

# Run initial tests
python -m pytest tests/ -v
python matryoshka.py --help
```

### Development Dependencies
```bash
# Add to requirements-dev.txt
pytest>=7.0.0
pytest-cov>=4.0.0
black>=22.0.0
flake8>=5.0.0
mypy>=0.991
pre-commit>=2.20.0
bandit>=1.7.0           # Security linting
safety>=2.0.0           # Dependency vulnerability scanning
```

## 🎨 Code Style Guidelines

### Python Code Standards
```python
# Use Black for code formatting
black matryoshka.py tests/

# Follow PEP 8 style guide
flake8 matryoshka.py --max-line-length=100

# Type hints for all functions
def log_finding(self, layer: str, artifact_type: str, location: str, 
               description: str, evidence_hash: str = "", 
               depth: int = 1) -> None:
    """Log a nested artifact finding to database"""
    pass

# Comprehensive docstrings
def analyze_registry_artifacts(self) -> int:
    """
    Analyze Windows registry for ephemeral communication artifacts.
    
    Scans registry keys commonly used by malware for persistence,
    communication configuration, and anti-forensics techniques.
    
    Returns:
        int: Number of registry artifacts discovered
        
    Raises:
        RegistryError: If registry access fails
        PermissionError: If insufficient privileges
    """
    pass
```

### Security-First Coding
```python
# Input validation for all user inputs
def validate_file_path(file_path: str) -> bool:
    """Validate file path to prevent directory traversal"""
    if not file_path or '..' in file_path:
        return False
    return Path(file_path).resolve().exists()

# Safe database operations
def safe_db_execute(self, query: str, params: tuple) -> None:
    """Execute database query with proper error handling"""
    try:
        with sqlite3.connect(self.db_path, timeout=10) as conn:
            conn.execute(query, params)
            conn.commit()
    except sqlite3.Error as e:
        logger.error(f"Database error: {e}")
        raise

# Secure file operations
def secure_hash_file(self, file_path: str) -> str:
    """Securely hash file contents"""
    if not self.validate_file_path(file_path):
        raise ValueError("Invalid file path")
    
    # Prevent hash collision attacks
    hash_obj = hashlib.sha256()
    with open(file_path, 'rb') as f:
        while chunk := f.read(8192):
            hash_obj.update(chunk)
    return hash_obj.hexdigest()
```

## 🧪 Testing Standards

### Test Structure
```
tests/
├── unit/
│   ├── test_core_functionality.py
│   ├── test_database_operations.py
│   ├── test_artifact_detection.py
│   └── test_ioc_extraction.py
├── integration/
│   ├── test_full_analysis.py
│   ├── test_cross_platform.py
│   └── test_performance.py
├── fixtures/
│   ├── sample_artifacts/
│   ├── test_databases/
│   └── mock_data.py
└── conftest.py
```

### Writing Tests
```python
import pytest
from unittest.mock import patch, MagicMock
from matryoshka import Matryoshka

class TestArtifactDetection:
    def setup_method(self):
        """Setup test environment"""
        self.analyzer = Matryoshka(db_path=":memory:")  # In-memory DB for tests
        
    def test_surface_layer_detection(self, tmp_path):
        """Test surface layer artifact detection"""
        # Create test artifacts
        test_file = tmp_path / "suspicious.tmp"
        test_file.write_text("test data")
        
        # Mock the temp paths to include our test directory
        with patch.object(self.analyzer, 'config') as mock_config:
            mock_config.__getitem__.return_value = [str(tmp_path)]
            
            findings = self.analyzer.peek_surface_layer()
            assert findings > 0
            assert any(f['location'] == str(test_file) for f in self.analyzer.findings)
    
    @pytest.mark.skipif(os.name == 'nt', reason="Unix-specific test")
    def test_memory_artifact_detection(self):
        """Test memory artifact detection on Unix systems"""
        with patch('os.path.exists') as mock_exists:
            mock_exists.return_value = True
            with patch('os.listdir') as mock_listdir:
                mock_listdir.return_value = ['sem.suspicious', '.hidden_mem']
                
                findings = self.analyzer.scan_memory_artifacts()
                assert findings == 2

    def test_ioc_extraction_confidence(self):
        """Test IOC confidence scoring"""
        self.analyzer.log_finding(
            "test_layer", "covert_channel", "127.0.0.1:31337",
            "Test finding", depth=6, ioc_confidence=0.95
        )
        
        ioc_report = self.analyzer.export_ioc_report()
        assert ioc_report['high_confidence_iocs'] == 1
```

### Performance Testing
```python
import time
import psutil
from memory_profiler import profile

def test_performance_large_filesystem():
    """Test performance with large number of files"""
    analyzer = Matryoshka()
    
    start_time = time.time()
    start_memory = psutil.Process().memory_info().rss / 1024 / 1024
    
    findings = analyzer.peek_surface_layer()
    
    end_time = time.time()
    end_memory = psutil.Process().memory_info().rss / 1024 / 1024
    
    # Performance assertions
    assert end_time - start_time < 60  # Should complete in under 1 minute
    assert end_memory - start_memory < 100  # Memory usage under 100MB
    assert findings >= 0  # Should not crash
```

## 🔒 Security Guidelines

### Secure Development Practices
```python
# Never execute user-provided code
def safe_metadata_parse(self, metadata_str: str) -> dict:
    """Safely parse metadata JSON"""
    try:
        # Use safe JSON parsing, never eval()
        return json.loads(metadata_str)
    except (json.JSONDecodeError, ValueError):
        logger.warning(f"Invalid metadata format: {metadata_str[:50]}")
        return {}

# Validate all external inputs
def validate_configuration(self, config: dict) -> bool:
    """Validate configuration parameters"""
    required_keys = ['max_file_size', 'recent_threshold_hours']
    
    if not all(key in config for key in required_keys):
        return False
        
    if not isinstance(config['max_file_size'], int) or config['max_file_size'] <= 0:
        return False
        
    return True

# Safe privilege checking
def check_admin_privileges(self) -> bool:
    """Safely check for administrative privileges"""
    try:
        if os.name == 'nt':
            import ctypes
            return ctypes.windll.shell32.IsUserAnAdmin()
        else:
            return os.geteuid() == 0
    except Exception as e:
        logger.debug(f"Privilege check failed: {e}")
        return False
```

### Malware Testing Safety
```python
# Guidelines for malware testing contributions:

# 1. ALWAYS use isolated virtual machines
# 2. NEVER commit actual malware samples to repository
# 3. Use hash references instead of actual files
# 4. Document all testing in controlled environments
# 5. Include cleanup procedures in test documentation

def test_with_malware_artifacts():
    """
    Test MATRYOSHKA against known malware artifacts
    
    SECURITY NOTE: This test requires controlled malware samples.
    Only run in isolated VM environments. Never commit samples to repo.
    """
    if not os.getenv('MALWARE_TESTING_ENV'):
        pytest.skip("Malware testing requires MALWARE_TESTING_ENV=true")
        
    # Test implementation with proper safety measures
    pass
```

## 📋 Pull Request Process

### Before Submitting
```bash
# Pre-submission checklist
□ Code follows style guidelines (run black, flake8)
□ All tests pass (pytest tests/ -v)
□ Security scan passes (bandit -r matryoshka.py)
□ Documentation updated for new features
□ CHANGELOG.md updated with changes
□ No sensitive data in commits (API keys, malware samples, etc.)
```

### PR Template
```markdown
## 🎯 Description
Brief description of changes and motivation.

## 🔄 Type of Change
- [ ] Bug fix (non-breaking change which fixes an issue)
- [ ] New feature (non-breaking change which adds functionality)
- [ ] Breaking change (fix or feature that causes existing functionality to change)
- [ ] Documentation update
- [ ] Security enhancement
- [ ] Performance improvement

## 🧪 Testing
- [ ] Unit tests added/updated
- [ ] Integration tests pass
- [ ] Manual testing completed
- [ ] Performance impact assessed

## 📚 Documentation
- [ ] Code comments updated
- [ ] README.md updated (if applicable)
- [ ] API documentation updated
- [ ] Usage examples added/updated

## 🔒 Security Considerations
- [ ] No sensitive data exposed
- [ ] Input validation implemented
- [ ] Privilege escalation considerations addressed
- [ ] Safe for production forensic environments

## 📷 Screenshots (if applicable)
[Add screenshots of new features or UI changes]

## ✅ Checklist
- [ ] My code follows the project's style guidelines
- [ ] I have performed a self-review of my code
- [ ] My changes generate no new warnings
- [ ] I have added tests that prove my fix is effective or that my feature works
- [ ] New and existing unit tests pass locally with my changes
```

## 🌟 Recognition

### Contribution Types
- **🏆 Core Contributors**: Major features, architecture improvements
- **🔍 Forensics Experts**: Detection logic, malware analysis insights
- **🐛 Bug Hunters**: Issue identification and resolution
- **📚 Documentation Masters**: Comprehensive documentation and tutorials
- **🧪 Test Engineers**: Comprehensive testing and validation
- **🌍 Community Leaders**: Help with community management and outreach

### Hall of Fame
Contributors will be recognized in:
- Project README.md
- Release notes and changelogs
- Conference presentations and research papers
- Social media acknowledgments as well!

## 🚫 Code of Conduct

### Our Standards
- **Professional Communication**: Respectful, inclusive, and constructive dialogue
- **Security First**: Never compromise on security practices or user safety
- **Ethical Use**: Contributions must support legitimate forensics and security research
- **Knowledge Sharing**: Help others learn and grow in the forensics community
- **Quality Focus**: Maintain high standards for code quality and documentation

### Unacceptable Behavior
- Malicious code or intentional backdoors
- Sharing actual malware samples in the repository
- Harassment or discriminatory language
- Compromising user security or privacy
- Violating applicable laws or regulations

## 📞 Getting Help

### Communication Channels
- **GitHub Issues**: Bug reports and feature requests
- **GitHub Discussions**: General questions and community discussions
- **Security Issues and Further Inquiries**: matlabmatryoshka@gmail.com

## 🎓 Learning Resources

### Forensics & Malware Analysis
- [SANS Digital Forensics](https://www.sans.org/cyber-security-courses/digital-forensics/)
- [Malware Analysis Techniques](https://github.com/rshipp/awesome-malware-analysis)
- [NIST Cybersecurity Framework](https://www.nist.gov/cybersecurity/cybersecurity-framework)

### Python Development
- [Python Security Best Practices](https://python.org/dev/security/)
- [Testing with pytest](https://docs.pytest.org/)
- [Type Checking with mypy](https://mypy.readthedocs.io/)

### Open Source Contribution
- [GitHub Flow](https://guides.github.com/introduction/flow/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Semantic Versioning](https://semver.org/)

---

## 🙏 Thank You

Every contribution, no matter how small, makes MATRYOSHKA better and helps the global cybersecurity community. Thank you for being part of this project!

**Questions?** Don't hesitate to open a GitHub Discussion or reach out to the maintainers.

---

*"In the spirit of the Russian nesting doll, every contribution reveals new layers of possibility."* 🪆
