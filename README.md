# Employee Data Synchronization (SuccessFactors to S/4HANA) using SAP Integration Suite

## Project Overview

This project implements an automated employee data integration scenario using **SAP BTP Cloud Integration**. Organizations often maintain employee information across multiple SAP systems, making manual data transfer time-consuming and error-prone. This solution retrieves employee data from **SAP SuccessFactors**, maps it to the **SAP S/4HANA** structure, creates employee records in S/4HANA, converts the processed data from XML to CSV, and distributes the output through **SFTP** and **Email**. An Exception Subprocess handles integration failures and notifies administrators automatically.

## Integration Flow

**I-Flow 1 — Main Process**

```
Timer Start Event
   ↓
Fetch Employee Data (SuccessFactors OData)
   ↓
Message Mapping — SF to S/4HANA
   ↓
Create Record in S/4HANA (Request-Reply)
   ↓
Mapping — Required Fields Only
   ↓
XML to CSV Converter
   ↓
Send CSV File to SFTP Server
   ↓
Send Success Notification (Mail)
```

**I-Flow 2 — Exception Subprocess**

```
Error Start Event
   ↓
Exception Message Body (Content Modifier)
   ↓
Send Error Notification (Mail)
```

## Technologies Used

- SAP BTP
- SAP Integration Suite – Cloud Integration
- SAP SuccessFactors (OData)
- SAP S/4HANA (OData)
- SFTP Adapter
- Message Mapping
- XML to CSV Converter
- Content Modifier
- Request-Reply
- Mail / SMTP Adapter
- Exception Subprocess
- CPI Security Material (credential management)

## Key Features

- Scheduled (Timer-based) automatic trigger — no manual execution required
- Employee data retrieval from SAP SuccessFactors
- Source-to-target field mapping (Employee ID, Name, Country, City, etc.)
- Automated employee record creation in S/4HANA
- Filtering to required fields only after S/4HANA processing
- XML to CSV conversion for file-based reporting
- Secure file delivery via SFTP
- Distribution via Sequential Multicast (SFTP + Email in parallel)
- Exception Subprocess for centralized error handling
- Automated error and success email notifications
- End-to-end monitoring via the CPI Monitor

## Methodology

Delivered using **Agile Scrum** across 5 sprints:

| Sprint | Focus |
|---|---|
| Sprint 1 | Environment setup and base iFlow creation |
| Sprint 2 | SuccessFactors data retrieval and mapping |
| Sprint 3 | S/4HANA integration, record creation, and field mapping |
| Sprint 4 | XML to CSV conversion and file distribution |
| Sprint 5 | Exception handling, testing, and deployment |

## Testing

The integration was validated with structured, evidence-backed test cases covering both positive and negative scenarios.

### TC01 — Standard Execution (Happy Path)
Flow triggered via the configured SuccessFactors OData query. S/4HANA returned `201 Created`, the SFTP server received the mapped CSV file, and a success notification email was delivered. **Result: Successful**

### TC02 — Duplicate Key Rejection (Negative Test)
Triggered with a Participant ID that already existed in S/4HANA. S/4HANA rejected the payload with `HTTP 400 Bad Request`; the Exception Subprocess caught the fault and emailed the administrator with error details and timestamp. **Result: Handled**

### TC03 — Network Timeout / Bad Gateway (Negative Test)
Triggered while the S/4HANA backend was temporarily unavailable. The Request-Reply call failed with `HTTP 502 Bad Gateway`; the Exception Subprocess routed the exact error to the administrator. **Result: Handled**

### TC04 — XML to CSV Converter: Invalid File Path (Negative Test)
Triggered with an incorrectly configured Path to Source on the XML to CSV Converter. The converter failed before reaching the SFTP send step; the path was corrected and the CSV output was revalidated. **Result: Handled**

## Defects & Resolutions

| ID | Defect | Resolution |
|---|---|---|
| BUG01 | Incorrect employee field mapping | Corrected field mapping |
| BUG02 | Data format mismatch | Corrected data transformation |
| BUG03 | Invalid employee payload | Added validation and error handling |
| BUG04 | CSV output format issue | Corrected XML to CSV configuration |
| BUG05 | Integration exception not notified | Configured Exception Subprocess and email |
| BUG06 | Output channel issue | Corrected receiver configuration |

## Repository Structure

```
├── Artifact/         → Exported iFlow (.iflw / ZIP bundle)
├── Documentation/     → Capstone project documentation (Word/PDF)
├── Presentation/      → Project presentation (PPTX)
├── Screenshots/        → Architecture, mapping, and test evidence
├── Version-History/    → Iteration notes and change log
└── README.md
```

## Project Evidence

Screenshots covering the integration flow design, adapter configurations, message mappings, and test execution (successful and error scenarios) are available in the `Screenshots` folder. Full technical write-up is available in the `Documentation` folder, and the summary deck is in `Presentation`.

## Conclusion

This integration automates employee data synchronization from SAP SuccessFactors to SAP S/4HANA and securely delivers a payroll-ready CSV file to the SFTP server. Every adapter authenticates through centrally managed Security Material, so no credential is exposed in the design artifact. Built-in exception handling protects the process end-to-end — duplicate-key rejections, backend unavailability, and converter configuration errors were each reproduced, diagnosed, and resolved during testing.

## Author

**Nalumachu Rahul Sai**
P-User ID: P2012786030
Client: GlobalTech Ltd (Training Scenario)
