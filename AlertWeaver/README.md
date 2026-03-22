# Alert Weaver

## Project Overview

Alert Weaver is an alert deduplication and clustering engine designed for cybersecurity teams. It ingests raw security alerts from various sources in CSV or JSON formats, normalizes inconsistent fields across different vendors, removes exact and near-duplicate alerts, and clusters related alerts into meaningful incidents based on shared entities and time windows. The output is a set of grouped incidents, reducing alert fatigue and improving incident response efficiency.

## Problem Statement

Security Operations Centers (SOCs) and cybersecurity teams are often overwhelmed by a high volume of alerts from multiple security tools and vendors. These alerts frequently contain duplicates, near-duplicates, or related events that should be grouped together. Without proper deduplication and clustering, analysts waste time sifting through repetitive or fragmented information, leading to missed threats or delayed responses. Alert Weaver addresses this by providing a standardized, automated way to process and consolidate alerts into actionable incidents.

## Why This Project Matters for SOCs / Security Teams

- **Reduces Alert Fatigue**: By deduplicating and clustering alerts, teams can focus on genuine incidents rather than noise.
- **Improves Efficiency**: Automates the normalization and grouping process, saving time and reducing manual effort.
- **Enhances Incident Response**: Provides a clear view of related events, enabling faster and more accurate threat analysis.
- **Vendor Agnostic**: Works with alerts from any source, normalizing inconsistencies for unified processing.
- **Scalable Solution**: Can handle large volumes of alerts, making it suitable for growing SOCs.

## Core MVP Goal

The MVP of Alert Weaver aims to demonstrate a functional alert processing pipeline that ingests, normalizes, deduplicates, and clusters alerts into incidents. Success is measured by the ability to process sample datasets and produce meaningful groupings that reduce alert volume while preserving critical information.

## Planned Features

- **Ingestion**: Support for CSV and JSON input formats.
- **Normalization**: Convert vendor-specific fields to a canonical schema.
- **Deduplication**: Identify and suppress exact duplicates.
- **Clustering**: Group related alerts based on shared entities (e.g., IP addresses, usernames) and time windows.
- **Summarization**: Generate incident summaries with explanations of groupings.
- **Export**: Output grouped incidents and mappings in JSON or CSV format.
- **CLI Interface**: Command-line tool for processing alerts.
- **Future UI**: Web-based interface for visualization and management (post-MVP).

## In Scope vs Out of Scope for MVP

### In Scope
- Basic ingestion from CSV and JSON files.
- Field normalization to a predefined canonical schema.
- Exact deduplication based on key fields.
- Rule-based clustering using shared entities and configurable time windows.
- Incident summarization with grouping explanations.
- Export of results to JSON/CSV.
- CLI for running the pipeline.

### Out of Scope
- Advanced ML-based clustering or anomaly detection.
- Integration with live SIEM systems or APIs.
- Real-time processing.
- User authentication or multi-user support.
- Graph-based or complex clustering algorithms.
- Database storage (beyond in-memory for MVP).
- UI development.

## Proposed Input Formats

- **CSV**: Tabular format with headers. Expected columns may vary by vendor but should include fields like timestamp, source IP, destination IP, alert type, severity, etc.
- **JSON**: Array of objects, each representing an alert. Flexible structure to accommodate different schemas.

Example CSV:
```
timestamp,source_ip,dest_ip,alert_type,severity,message
2023-10-01T10:00:00Z,192.168.1.1,10.0.0.1,Brute Force,High,Failed login attempts
```

Example JSON:
```json
[
  {
    "timestamp": "2023-10-01T10:00:00Z",
    "source_ip": "192.168.1.1",
    "dest_ip": "10.0.0.1",
    "alert_type": "Brute Force",
    "severity": "High",
    "message": "Failed login attempts"
  }
]
```

## Canonical Normalized Alert Fields

After normalization, all alerts will conform to a standard schema:

- `id`: Unique identifier for the alert.
- `timestamp`: ISO 8601 formatted timestamp.
- `source_ip`: Source IP address (normalized).
- `dest_ip`: Destination IP address (normalized).
- `alert_type`: Standardized alert category (e.g., "Brute Force", "Malware Detection").
- `severity`: Standardized severity level (e.g., "Low", "Medium", "High", "Critical").
- `message`: Human-readable description.
- `entities`: List of extracted entities (e.g., IPs, domains, usernames).
- `vendor`: Original vendor name.
- `raw_data`: Original alert data for reference.

## Expected Output Structure

The output will include:
- **Grouped Incidents**: JSON array of incidents, each containing:
  - `incident_id`: Unique ID for the incident.
  - `alerts`: List of normalized alert IDs in the incident.
  - `summary`: Brief description of the incident.
  - `severity`: Highest severity among alerts.
  - `start_time`: Earliest timestamp.
  - `end_time`: Latest timestamp.
  - `entities`: Shared entities across alerts.
  - `grouping_reason`: Explanation of why alerts were grouped.
- **Mappings**: JSON object mapping original alert IDs to incident IDs.

Example Output:
```json
{
  "incidents": [
    {
      "incident_id": "INC-001",
      "alerts": ["alert-1", "alert-2"],
      "summary": "Brute force attack on server",
      "severity": "High",
      "start_time": "2023-10-01T10:00:00Z",
      "end_time": "2023-10-01T10:15:00Z",
      "entities": ["192.168.1.1", "10.0.0.1"],
      "grouping_reason": "Shared source and destination IPs within 15-minute window"
    }
  ],
  "mappings": {
    "original-alert-1": "INC-001",
    "original-alert-2": "INC-001"
  }
}
```

## High-Level Architecture

1. **Ingestion Module**: Reads and parses input files (CSV/JSON).
2. **Normalization Module**: Maps vendor fields to canonical schema.
3. **Deduplication Module**: Identifies and removes duplicates.
4. **Clustering Module**: Applies rules to group alerts into incidents.
5. **Summarization Module**: Generates incident summaries and explanations.
6. **Export Module**: Writes results to output files.
7. **CLI/App Entry Point**: Orchestrates the pipeline.

Data flows: Raw Input → Ingestion → Normalization → Deduplication → Clustering → Summarization → Export.

## Proposed Python Project Structure

```
AlertWeaver/
├── README.md
├── requirements.txt
├── app.py
├── data/
│   ├── raw/
│   ├── processed/
│   └── output/
├── alert_weaver/
│   ├── __init__.py
│   ├── config.py
│   ├── ingestion.py
│   ├── normalization.py
│   ├── deduplication.py
│   ├── clustering.py
│   ├── summarization.py
│   ├── export.py
│   └── utils.py
└── tests/
```

- `app.py`: Main CLI application.
- `alert_weaver/`: Core modules.
- `data/`: Directories for input, intermediate, and output data.
- `tests/`: Unit and integration tests.

## Suggested Tech Stack

- **Language**: Python 3.8+
- **Core Libraries**:
  - `pandas`: For data manipulation and CSV handling.
  - `pydantic`: For data validation and schema enforcement.
  - `click`: For CLI interface.
- **Testing**: `pytest` for unit tests.
- **Development**: VS Code with Python extension, virtualenv for environment management.

## Deduplication and Clustering Approach for MVP

- **Deduplication**: Compare alerts on key fields (e.g., timestamp, source_ip, dest_ip, alert_type). Use exact matching or fuzzy matching for near-duplicates.
- **Clustering**: Rule-based approach:
  - Group alerts sharing at least one entity (e.g., IP) within a configurable time window (e.g., 15 minutes).
  - Merge groups iteratively until no more overlaps.
  - Prioritize by severity and recency.

This approach is simple, interpretable, and effective for MVP without requiring ML.

## Main Risks / Challenges

- **Data Quality**: Inconsistent or missing fields in input data.
- **Clustering Accuracy**: Balancing over-grouping vs. under-grouping; tuning time windows and entity matching.
- **Performance**: Handling large datasets efficiently.
- **Vendor Variability**: Accommodating diverse alert schemas.
- **Testing**: Obtaining realistic test data without sensitive information.

## Development Phases

1. **Phase 1: Planning and Setup** (Current)
   - Project structure, documentation, and initial setup.
2. **Phase 2: Core Implementation**
   - Build ingestion, normalization, deduplication, and basic clustering.
3. **Phase 3: Integration and Testing**
   - Integrate modules, add CLI, write tests, and validate with sample data.
4. **Phase 4: Refinement and Demo**
   - Optimize performance, improve clustering logic, and prepare for demonstration.

## Future Enhancements

- **Advanced Clustering**: ML-based similarity scoring, graph algorithms.
- **Real-time Processing**: Streaming ingestion and incremental clustering.
- **UI**: Web app with Streamlit or FastAPI for visualization.
- **Database Integration**: SQLite or PostgreSQL for persistence.
- **API Endpoints**: REST API for integration with SIEM tools.
- **Alert Enrichment**: Add threat intelligence lookups.

## Local Development Setup

1. Clone the repository.
2. Create a virtual environment: `python -m venv venv`
3. Activate: `venv\Scripts\activate` (Windows) or `source venv/bin/activate` (Unix)
4. Install dependencies: `pip install -r requirements.txt`
5. Run tests: `pytest`
6. Run the app: `python app.py --help`

## How This Project Could Be Demonstrated in a Portfolio

- **Demo Video/Script**: Show processing a sample CSV of alerts, displaying before/after volumes, and explaining groupings.
- **Code Quality**: Highlight modular design, tests, and documentation.
- **Problem-Solving**: Discuss challenges in normalization and clustering, and how they were addressed.
- **Impact**: Emphasize real-world value for SOCs, with metrics like alert reduction percentage.
- **Scalability**: Note how the architecture supports future enhancements.

## MVP Success Statement

Version 1 is successful if it can ingest raw CSV/JSON alerts, normalize fields into a canonical schema, suppress exact duplicates, group related alerts into incidents using rules and time windows, explain why alerts were grouped, and export grouped incidents and raw-to-cluster mappings.