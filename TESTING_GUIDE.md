# Matryoshka Testing Guide

## Pre-Test Setup

### 1. Create Test Environment
```bash
# Create isolated test directory
mkdir matryoshka_test
cd matryoshka_test

# Set up test structure
mkdir -p test_artifacts/{temp,hidden,suspicious}
mkdir -p test_logs
```

### 2. Generate Test Artifacts
Create various artifacts that Matryoshka should detect:

```bash
# Recent temporary files
touch /tmp/recent_temp_file.tmp
touch /tmp/.hidden_temp_file
echo "test data" > /tmp/suspicious_comm_pipe.sock

# Files with suspicious patterns
touch /tmp/ipc_channel.tmp
touch /tmp/.backdoor_comm
echo "test" > /tmp/covert_channel.pipe

# Create files with unusual timestamps (simulate timestomping)
touch -t 202501010000 /tmp/timestomped_file.txt

# Windows-specific (if testing on Windows)
# These would be in %TEMP% directory instead
```

### 3. Create Test Configuration
```bash
cat > test_config.json << 'EOF'
{
  "max_file_size": 10485760,
  "recent_threshold_hours": 24,
  "hash_algorithm": "sha256",
  "max_workers": 2,
  "max_depth_analysis": 100,
  "suspicious_ports": [4444, 5555, 31337],
  "temp_paths_unix": ["/tmp", "/var/tmp"],
  "temp_paths_windows": ["%TEMP%", "%TMP%"]
}
EOF
```

## Basic Functionality Tests

### Test 1: Command Line Interface
```bash
# Test help output
python matryoshka_enhanced.py
python matryoshka_enhanced.py --help

# Expected: Should display usage information and feature list
```

### Test 2: Configuration Loading
```bash
# Test with custom config
python matryoshka_enhanced.py --analyze --config test_config.json --log-level DEBUG

# Expected: Should load configuration and start analysis
# Check logs for "Loaded configuration from test_config.json"
```

### Test 3: Database Creation
```bash
# Run basic analysis
python matryoshka.py --analyze --db test_analysis.db

# Verify database was created
ls -la test_analysis.db
sqlite3 test_analysis.db ".tables"

# Expected: Should show forensic_evidence, analysis_sessions, indicators tables
```

### Test 4: Surface Artifact Detection
```bash
# Create fresh temporary files
touch /tmp/test_artifact_$(date +%s).tmp
echo "test content" > /tmp/.hidden_test_file

# Run analysis immediately
python matryoshka_enhanced.py --analyze --log-level INFO

# Expected: Should detect the recently created files
```

## Advanced Testing Scenarios

### Test 5: Privilege Level Detection
```bash
# Test as regular user
python matryoshka_enhanced.py --analyze --log-level DEBUG

# Test with elevated privileges
sudo python matryoshka_enhanced.py --analyze --log-level DEBUG

# Expected: Should report different privilege levels and access capabilities
```

### Test 6: Threat Scoring System
```bash
# Create high-threat artifacts
touch /tmp/.suspicious_ipc_channel.sock
touch /tmp/covert_comm_pipe.tmp
echo "backdoor" > /tmp/.hidden_backdoor_comm

# Run analysis
python matryoshka_enhanced.py --analyze --export-report --output threat_test.json

# Check results
cat threat_test.json | python -m json.tool

# Expected: Should assign appropriate threat levels and confidence scores
```

### Test 7: Timestomp Detection
```bash
# Create files with suspicious timestamps
touch -t 197001010000 /tmp/epoch_timestamp.txt  # Unix epoch
touch -t 200001010000 /tmp/y2k_timestamp.txt    # Y2K
touch -t 203001010000 /tmp/future_timestamp.txt  # Future date

# Run analysis
python matryoshka_enhanced.py --analyze --log-level DEBUG

# Expected: Should detect timestomping indicators
```

### Test 8: Memory Artifact Detection (Linux/Unix)
```bash
# Create shared memory artifacts
echo "test_data" > /dev/shm/test_memory_artifact
touch /dev/shm/.hidden_memory_file
mkfifo /dev/shm/test_pipe 2>/dev/null || true

# Run analysis
python matryoshka_enhanced.py --analyze

# Expected: Should detect memory artifacts with appropriate scoring
```

### Test 9: Network Shadow Detection
```bash
# Start a test service on suspicious port (if available)
# This requires careful testing to avoid conflicts
python -c "
import socket
import time
try:
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.bind(('127.0.0.1', 31337))
    s.listen(1)
    print('Test service started on port 31337')
    time.sleep(30)  # Keep it running for analysis
except Exception as e:
    print(f'Could not start test service: {e}')
finally:
    s.close()
" &

# Run analysis while service is running
python matryoshka_enhanced.py --analyze
kill %1  # Stop background service

# Expected: Should detect suspicious network activity
```

## Validation Tests

### Test 10: Input Validation
```bash
# Test path traversal protection
python matryoshka_enhanced.py --analyze --config "../../../etc/passwd" 2>&1

# Test invalid database paths
python matryoshka_enhanced.py --analyze --db "invalid/../path/db.sqlite" 2>&1

# Expected: Should handle errors gracefully with security warnings
```

### Test 11: Report Export Functionality
```bash
# Generate findings and export
python matryoshka_enhanced.py --analyze --export-report --output detailed_test_report.json

# Validate JSON structure
python -c "
import json
with open('detailed_test_report.json', 'r') as f:
    data = json.load(f)
    print(f'Report contains {len(data[\"findings\"])} findings')
    print(f'Metadata: {data[\"metadata\"]}')
"

# Expected: Should create valid JSON report with proper structure
```

### Test 12: Database Integrity
```bash
# Run multiple analysis sessions
python matryoshka_enhanced.py --analyze --db integrity_test.db
python matryoshka_enhanced.py --analyze --db integrity_test.db

# Check database consistency
sqlite3 integrity_test.db "
SELECT COUNT(*) as sessions FROM analysis_sessions;
SELECT COUNT(*) as evidence FROM forensic_evidence;
SELECT COUNT(*) as indicators FROM indicators;
"

# Expected: Should maintain data integrity across sessions
```

## Performance Testing

### Test 13: Large Dataset Handling
```bash
# Create many test files
for i in {1..1000}; do
    touch /tmp/test_file_$i.tmp
done

# Test with depth limits
python matryoshka_enhanced.py --analyze --config test_config.json --log-level INFO

# Clean up
rm /tmp/test_file_*.tmp

# Expected: Should respect max_depth_analysis limits and complete without errors
```

### Test 14: Concurrent Processing
```bash
# Test with different worker counts
python matryoshka_enhanced.py --analyze --log-level DEBUG

# Monitor system resources during analysis
htop  # or top, depending on system

# Expected: Should utilize multiple threads efficiently
```

## Error Handling Tests

### Test 15: Permission Errors
```bash
# Create files with restricted permissions
sudo touch /tmp/restricted_file.tmp
sudo chmod 000 /tmp/restricted_file.tmp

# Run analysis
python matryoshka_enhanced.py --analyze --log-level DEBUG

# Clean up
sudo rm /tmp/restricted_file.tmp

# Expected: Should handle permission errors gracefully
```

### Test 16: Interrupted Analysis
```bash
# Start analysis and interrupt
python matryoshka_enhanced.py --analyze &
sleep 5
kill -INT %1  # Send SIGINT

# Expected: Should shut down gracefully with cleanup message
```

## Results Validation

After each test, verify:

1. **No crashes or unhandled exceptions**
2. **Appropriate log messages at specified levels**
3. **Database files created with correct permissions (600)**
4. **Evidence records have proper structure and IDs**
5. **Threat scores are reasonable for created artifacts**
6. **Reports are valid JSON with expected schema**
7. **Temporary files are cleaned up**

## Cleanup After Testing

```bash
# Remove test artifacts
rm -f /tmp/test_*.tmp /tmp/.hidden_* /tmp/suspicious_* /tmp/*comm* /tmp/*pipe*
rm -f /dev/shm/test_* 2>/dev/null || true

# Remove test databases and reports
rm -f *.db *.json test_config.json

# Remove test directories
rm -rf matryoshka_test test_logs
```

## Expected Baseline Results

A successful test should produce:
- At least 5-10 surface artifacts detected
- Threat levels ranging from LOW to HIGH
- Confidence scores between 0.3-0.9
- Valid database with foreign key constraints
- JSON reports with proper evidence chain of custody
- No security validation errors in logs

This testing approach validates both the core functionality and the security enhancements of Matryoshka v2.0.
