# MATRYOSHKA Database Documentation

MATRYOSHKA uses SQLite as its embedded database engine to store forensic findings, analysis sessions, and extracted Indicators of Compromise (IOCs). The database is automatically created and managed, requiring no external database server or complex configuration.

## Database Schema

### Core Tables

#### `nested_findings`
Primary table storing all discovered artifacts across analysis layers.

```sql
CREATE TABLE nested_findings (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    timestamp TEXT NOT NULL,
    layer TEXT NOT NULL,
    artifact_type TEXT NOT NULL,
    location TEXT NOT NULL,
    description TEXT,
    evidence_hash TEXT,
    depth_score INTEGER,
    metadata TEXT,
    file_size INTEGER,
    permissions TEXT,
    ioc_confidence REAL DEFAULT 0.0,
    threat_level TEXT DEFAULT 'LOW'
);
```

**Field Descriptions:**
- `id`: Unique identifier for each finding
- `timestamp`: ISO format timestamp when artifact was discovered
- `layer`: Analysis layer (surface_layer, deletion_layer, process_layer, etc.)
- `artifact_type`: Type of artifact (recent_temp_artifact, covert_channel_trace, etc.)
- `location`: File path, network endpoint, or registry key location
- `description`: Human-readable description of the finding
- `evidence_hash`: SHA-256 hash of associated file (if applicable)
- `depth_score`: Threat depth score (1-6, higher = more suspicious)
- `metadata`: JSON-encoded additional information
- `file_size`: Size of associated file in bytes
- `permissions`: File permissions in octal format
- `ioc_confidence`: ML-based confidence score (0.0-1.0)
- `threat_level`: Classification (LOW, MEDIUM, HIGH)

#### `analysis_sessions`
Tracks individual analysis runs and session metadata.

```sql
CREATE TABLE analysis_sessions (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    start_time TEXT NOT NULL,
    end_time TEXT,
    total_layers INTEGER DEFAULT 0,
    deepest_layer INTEGER DEFAULT 0,
    privilege_level TEXT,
    system_info TEXT
);
```

**Field Descriptions:**
- `id`: Unique session identifier
- `start_time`: Analysis start timestamp (ISO format)
- `end_time`: Analysis completion timestamp
- `total_layers`: Total number of artifacts found
- `deepest_layer`: Deepest analysis layer reached (1-6)
- `privilege_level`: Execution privilege level (admin/user)
- `system_info`: JSON-encoded system information

#### `iocs`
Extracted Indicators of Compromise for threat intelligence.

```sql
CREATE TABLE iocs (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    ioc_type TEXT NOT NULL,
    value TEXT NOT NULL,
    confidence REAL,
    first_seen TEXT,
    last_seen TEXT,
    source_finding_id INTEGER,
    FOREIGN KEY (source_finding_id) REFERENCES nested_findings (id)
);
```

**Field Descriptions:**
- `id`: Unique IOC identifier
- `ioc_type`: IOC category (file_path, network_endpoint, registry_key, etc.)
- `value`: IOC value (path, IP:port, registry key, etc.)
- `confidence`: Confidence score (0.0-1.0)
- `first_seen`: First detection timestamp
- `last_seen`: Most recent detection timestamp
- `source_finding_id`: Reference to originating finding

## Database Operations

### Automatic Creation
```python
# Database is automatically created on first run
matryoshka = Matryoshka("investigation.db")
# Creates database file and initializes schema
```

### Manual Database Inspection
```bash
# Connect to database
sqlite3 matryoshka.db

# View schema
.schema

# List all tables
.tables

# View recent findings
SELECT timestamp, threat_level, artifact_type, location 
FROM nested_findings 
ORDER BY timestamp DESC 
LIMIT 10;

# Count findings by threat level
SELECT threat_level, COUNT(*) as count 
FROM nested_findings 
GROUP BY threat_level;

# Export high-confidence IOCs
SELECT ioc_type, value, confidence 
FROM iocs 
WHERE confidence > 0.8 
ORDER BY confidence DESC;
```

## Database Queries for Analysis

### Security Analysis Queries

#### High-Priority Threats
```sql
-- Find critical threats requiring immediate attention
SELECT timestamp, artifact_type, location, description, depth_score
FROM nested_findings 
WHERE threat_level = 'HIGH' 
ORDER BY depth_score DESC, timestamp DESC;
```

#### Covert Communication Analysis
```sql
-- Identify potential covert communication channels
SELECT location, description, ioc_confidence
FROM nested_findings 
WHERE artifact_type LIKE '%covert%' 
   OR artifact_type LIKE '%communication%'
   OR location LIKE '%sock%'
   OR location LIKE '%pipe%'
ORDER BY ioc_confidence DESC;
```

#### Timeline Analysis
```sql
-- Create timeline of suspicious activities
SELECT 
    DATE(timestamp) as date,
    COUNT(*) as artifacts,
    COUNT(CASE WHEN threat_level = 'HIGH' THEN 1 END) as high_threats,
    COUNT(CASE WHEN threat_level = 'MEDIUM' THEN 1 END) as medium_threats
FROM nested_findings 
GROUP BY DATE(timestamp)
ORDER BY date DESC;
```

#### Layer Distribution Analysis
```sql
-- Analyze artifact distribution across layers
SELECT 
    layer,
    COUNT(*) as total_artifacts,
    AVG(depth_score) as avg_depth,
    MAX(depth_score) as max_depth,
    COUNT(CASE WHEN threat_level = 'HIGH' THEN 1 END) as high_threats
FROM nested_findings 
GROUP BY layer
ORDER BY max_depth DESC;
```

### IOC Intelligence Queries

#### Network IOCs
```sql
-- Extract network-based indicators
SELECT DISTINCT value, confidence, first_seen
FROM iocs 
WHERE ioc_type = 'network_endpoint'
  AND confidence > 0.7
ORDER BY confidence DESC;
```

#### File System IOCs
```sql
-- Extract file system indicators
SELECT DISTINCT value, confidence, first_seen
FROM iocs 
WHERE ioc_type = 'file_path'
  AND confidence > 0.6
ORDER BY first_seen DESC;
```

#### Trending IOCs
```sql
-- Find IOCs appearing in multiple sessions
SELECT 
    value, 
    ioc_type, 
    COUNT(DISTINCT source_finding_id) as frequency,
    AVG(confidence) as avg_confidence,
    MIN(first_seen) as first_occurrence,
    MAX(last_seen) as last_occurrence
FROM iocs 
GROUP BY value, ioc_type
HAVING frequency > 1
ORDER BY frequency DESC, avg_confidence DESC;
```

## Advanced Database Usage

### Programmatic Access
```python
import sqlite3
import json
from datetime import datetime, timedelta

def get_recent_high_threats(db_path, hours=24):
    """Retrieve high-threat artifacts from last N hours"""
    conn = sqlite3.connect(db_path)
    cutoff = (datetime.now() - timedelta(hours=hours)).isoformat()
    
    cursor = conn.cursor()
    cursor.execute('''
        SELECT artifact_type, location, description, depth_score
        FROM nested_findings 
        WHERE threat_level = 'HIGH' 
          AND timestamp > ?
        ORDER BY depth_score DESC
    ''', (cutoff,))
    
    results = cursor.fetchall()
    conn.close()
    return results

def export_iocs_for_siem(db_path, min_confidence=0.8):
    """Export IOCs in SIEM-friendly format"""
    conn = sqlite3.connect(db_path)
    cursor = conn.cursor()
    
    cursor.execute('''
        SELECT ioc_type, value, confidence, first_seen
        FROM iocs 
        WHERE confidence >= ?
        ORDER BY confidence DESC
    ''', (min_confidence,))
    
    iocs = []
    for row in cursor.fetchall():
        iocs.append({
            'type': row[0],
            'value': row[1],
            'confidence': row[2],
            'first_seen': row[3],
            'source': 'MATRYOSHKA'
        })
    
    conn.close()
    return iocs
```

### Database Maintenance
```python
def cleanup_old_sessions(db_path, days=30):
    """Clean up analysis sessions older than specified days"""
    conn = sqlite3.connect(db_path)
    cutoff = (datetime.now() - timedelta(days=days)).isoformat()
    
    cursor = conn.cursor()
    
    # Delete old sessions and associated findings
    cursor.execute('DELETE FROM analysis_sessions WHERE start_time < ?', (cutoff,))
    cursor.execute('DELETE FROM nested_findings WHERE timestamp < ?', (cutoff,))
    
    # Vacuum database to reclaim space
    cursor.execute('VACUUM')
    
    conn.commit()
    conn.close()
```

## Database Best Practices

### Performance Optimization
```sql
-- Create indexes for common queries
CREATE INDEX idx_findings_timestamp ON nested_findings(timestamp);
CREATE INDEX idx_findings_threat_level ON nested_findings(threat_level);
CREATE INDEX idx_findings_layer ON nested_findings(layer);
CREATE INDEX idx_iocs_confidence ON iocs(confidence);
CREATE INDEX idx_iocs_type ON iocs(ioc_type);
```

### Backup and Recovery
```bash
# Backup database
cp matryoshka.db matryoshka_backup_$(date +%Y%m%d).db

# Compress backup
gzip matryoshka_backup_$(date +%Y%m%d).db

# Restore from backup
gunzip matryoshka_backup_20241201.db.gz
cp matryoshka_backup_20241201.db matryoshka.db
```

### Multi-Investigation Management
```bash
# Separate databases for different investigations
python3 matryoshka.py --open --db case_001_baseline.db
python3 matryoshka.py --open --db case_001_post_incident.db
python3 matryoshka.py --open --db case_002_threat_hunt.db

# Merge findings from multiple databases
sqlite3 merged_investigation.db "
ATTACH 'case_001_baseline.db' as db1;
ATTACH 'case_001_post_incident.db' as db2;
INSERT INTO nested_findings SELECT * FROM db1.nested_findings;
INSERT INTO nested_findings SELECT * FROM db2.nested_findings;
"
```

## Database File Locations

### Default Locations
- **Current Directory**: `./matryoshka.db`
- **Custom Path**: Specified via `--db` parameter
- **Investigation Structure**:
  ```
  investigations/
  ├── case_001/
  │   ├── baseline.db
  │   ├── post_incident.db
  │   └── iocs_export.json
  ├── case_002/
  │   └── threat_hunt.db
  └── templates/
      └── config.json
  ```

### Database Security
```bash
# Set appropriate permissions
chmod 600 matryoshka.db  # Owner read/write only

# Encrypt sensitive databases (using SQLCipher)
# Note: Requires SQLCipher installation
sqlite3 encrypted.db "PRAGMA key = 'your-encryption-key';"
```

## Troubleshooting

### Common Issues

#### Database Locked
```python
# Handle database locking issues
import time
import sqlite3

def safe_db_operation(db_path, operation):
    max_retries = 5
    for attempt in range(max_retries):
        try:
            conn = sqlite3.connect(db_path, timeout=10)
            return operation(conn)
        except sqlite3.OperationalError as e:
            if "database is locked" in str(e) and attempt < max_retries - 1:
                time.sleep(1)
                continue
            raise
        finally:
            if 'conn' in locals():
                conn.close()
```

#### Corrupted Database Recovery
```bash
# Check database integrity
sqlite3 matryoshka.db "PRAGMA integrity_check;"

# Repair corrupted database
sqlite3 corrupted.db ".recover" | sqlite3 recovered.db

# Export data from corrupted database
sqlite3 corrupted.db ".dump" > backup.sql
sqlite3 new.db < backup.sql
```

## Database Size Management

### Monitoring Database Growth
```sql
-- Check database size and statistics
SELECT 
    COUNT(*) as total_findings,
    MIN(timestamp) as earliest_finding,
    MAX(timestamp) as latest_finding,
    COUNT(DISTINCT layer) as active_layers
FROM nested_findings;

-- Check table sizes
SELECT 
    name,
    COUNT(*) as row_count
FROM sqlite_master 
WHERE type='table'
GROUP BY name;
```

### Archival Strategy
```python
def archive_old_findings(db_path, archive_path, days=90):
    """Archive findings older than specified days"""
    import shutil
    
    # Create archive database
    shutil.copy2(db_path, archive_path)
    
    # Remove old data from active database
    conn = sqlite3.connect(db_path)
    cutoff = (datetime.now() - timedelta(days=days)).isoformat()
    
    cursor = conn.cursor()
    cursor.execute('DELETE FROM nested_findings WHERE timestamp < ?', (cutoff,))
    cursor.execute('DELETE FROM iocs WHERE first_seen < ?', (cutoff,))
    cursor.execute('VACUUM')
    
    conn.commit()
    conn.close()
```

---

## Database Support

For database-related issues:
- Check SQLite version: `sqlite3 --version`
- Verify Python sqlite3 module: `python3 -c "import sqlite3; print(sqlite3.version)"`
- Review database integrity: `sqlite3 matryoshka.db "PRAGMA integrity_check;"`

The MATRYOSHKA database is designed to be self-managing and requires minimal maintenance while providing powerful forensic analysis capabilities.
