# Project: Burp Suite Pentest Activity Extension

## 1. Project Overview

Develop a fully functional Burp Suite Extension named **Pentest Activity**.

The extension must analyze HTTP traffic captured by Burp Suite and reconstruct penetration testing activities based on the requests and responses generated during security assessments.

The primary objective is to help penetration testers understand, track, search, and summarize their testing activities over time without manually documenting every action.

The extension must be able to answer questions such as:

- What activities did I perform today?
- What did I test on a specific date?
- What security testing activities did I perform between two dates?
- How many SQL Injection tests did I perform?
- Which endpoints did I test for IDOR?
- What were my overall activities during the entire penetration testing engagement?
- What endpoints and parameters were tested most frequently?
- What techniques and payloads did I use?

This extension must work directly inside Burp Suite and must not require an external web application.

## 2. Technology Stack

Use the following technologies:

- Programming Language: Java.
- Extension API: Official PortSwigger Montoya API.
- UI Framework: Java Swing.
- Database: SQLite.
- Database Access: JDBC.
- Build System: Maven.
- Packaging: Runnable extension JAR.
- Java Version: Use a version compatible with the selected Burp Montoya API release.

Use the official Montoya API documentation:
https://portswigger.github.io/burp-extensions-montoya-api/

The extension must be compatible with Burp Suite Professional and Community Edition wherever the required API capabilities are available.

Do not use deprecated Extender APIs or legacy IBurpExtender implementations.

## 3. Main Architecture

Implement the extension using a modular architecture.

Suggested package structure:

com.pentestactivity
├── PentestActivityExtension.java
├── core
│   ├── ExtensionInitializer.java
│   ├── ActivityManager.java
│   ├── TrafficCollector.java
│   └── ProjectManager.java
├── analyzer
│   ├── ActivityAnalyzer.java
│   ├── ActivityClassifier.java
│   ├── PayloadDetector.java
│   ├── PatternMatcher.java
│   └── ConfidenceCalculator.java
├── database
│   ├── DatabaseManager.java
│   ├── RequestRepository.java
│   ├── ActivityRepository.java
│   ├── ProjectRepository.java
│   └── DatabaseMigration.java
├── model
│   ├── HttpRequestRecord.java
│   ├── ActivityRecord.java
│   ├── Project.java
│   └── ActivityCategory.java
├── ui
│   ├── MainTab.java
│   ├── DashboardPanel.java
│   ├── TimelinePanel.java
│   ├── ActivityExplorerPanel.java
│   ├── ActivityDetailPanel.java
│   ├── RequestViewerPanel.java
│   ├── SettingsPanel.java
│   └── components
└── export
    ├── MarkdownExporter.java
    ├── CSVExporter.java
    └── JSONExporter.java

You may adjust the structure when necessary, but maintain clear separation between traffic collection, analysis, persistence, and presentation.

## 4. Traffic Collection

Integrate directly with Burp Suite using the Montoya API.

The extension must collect HTTP requests and responses from the available Burp HTTP history.

Requirements:

1. Retrieve existing HTTP history when the extension is initialized.
2. Monitor newly captured HTTP traffic.
3. Store relevant request and response metadata.
4. Capture original request timestamps whenever available.
5. Capture HTTP method, URL, hostname, path, query parameters, headers, request body, response status code, and response length.
6. Preserve the association between each stored record and its original Burp request information.
7. Avoid processing the same HTTP history item multiple times.
8. Support incremental collection without reprocessing the entire history unnecessarily.

Important:

Use actual timestamps provided by Burp when available.

Do not use the time when the extension processes a request as a replacement for its original timestamp.

If the API does not expose a required historical timestamp or identifier, implement an explicit fallback and document the limitation.

Do not generate additional HTTP requests to target applications.

The extension must be passive and must not behave like an active vulnerability scanner.

## 5. Activity Classification Engine

Develop a rule-based activity detection engine.

The engine must identify potential penetration testing activities based on HTTP request and response patterns.

Initial supported categories:

### SQL Injection
Detect patterns such as:

- Single quote and double quote injection.
- OR-based payloads.
- UNION-based payloads.
- Boolean-based SQL Injection.
- Time-based SQL Injection.
- Database error messages.
- SQL syntax manipulation.
- Database-specific functions.

### IDOR / BOLA
Detect potential object reference manipulation:

- Changes to user_id.
- Changes to account_id.
- Changes to customer_id.
- Changes to transaction_id.
- Changes to document_id.
- Changes to resource identifiers.
- Sequential identifier manipulation.
- Access to different object references.

Do not automatically classify every changing ID as IDOR. Consider request context and evidence.

### Authentication Testing
Detect:

- JWT manipulation.
- Token modification.
- Missing authentication tokens.
- Invalid token testing.
- Expired token testing.
- Authorization header manipulation.
- Login endpoint testing.
- Session manipulation.

### Authorization Testing
Detect:

- Role manipulation.
- Privilege escalation attempts.
- Access to restricted endpoints.
- User privilege changes.
- Cross-user access attempts.
- Administrative endpoint access.

### Cross-Site Scripting
Detect:

- Script injection payloads.
- HTML injection.
- Event handler injection.
- Reflected XSS testing.
- DOM-related injection patterns where observable.

### Server-Side Request Forgery
Detect:

- URL parameter manipulation.
- Internal IP addresses.
- Localhost references.
- Cloud metadata addresses.
- External callback URLs.
- Protocol manipulation.

### Business Logic Testing
Detect:

- Negative price manipulation.
- Quantity manipulation.
- Discount manipulation.
- Payment amount manipulation.
- Workflow state manipulation.
- Approval status manipulation.
- Transaction parameter manipulation.
- Future or past date manipulation.

### Input Validation Testing
Detect:

- Unexpected data types.
- Null values.
- Empty values.
- Oversized input.
- Boundary values.
- Special characters.
- Invalid formats.
- Parameter type manipulation.

### Fuzzing
Detect:

- Repeated requests to the same endpoint.
- Multiple payload variations.
- Parameter mutation.
- High-volume testing patterns.

### Reconnaissance
Detect:

- Endpoint discovery.
- Directory enumeration.
- API endpoint discovery.
- Parameter discovery.
- HTTP method discovery.

The classification engine must be extensible so new categories and detection rules can be added without modifying the entire application.

## 6. Activity Correlation and Grouping

This is one of the most important components.

Do not display every HTTP request as an independent activity.

Group related requests into logical testing activities.

Example:

A tester sends 50 requests to:

POST /api/user/search

With different SQL Injection payloads.

The extension should display:

Activity: SQL Injection Testing

Endpoint: POST /api/user/search

Date: 20 November 2026

Total Requests: 50

Detected Techniques:
- Boolean-based
- Error-based
- Time-based

The grouping engine should consider:

- Target hostname.
- Endpoint.
- HTTP method.
- Tested parameter.
- Payload characteristics.
- Detection category.
- Request timestamps.
- Similarity between requests.

Allow multiple requests to belong to the same activity.

Allow manual merging and splitting of activities.

Do not merge unrelated testing activities just because they target the same endpoint.

## 7. Main Dashboard

Create a dedicated Burp Suite tab named:

Pentest Activity

The dashboard should provide an overview of the current project.

Display:

- Total HTTP requests analyzed.
- Total testing activities.
- Number of active testing days.
- Number of detected testing categories.
- First and last activity timestamps.
- Most frequently tested endpoints.
- Most frequently detected testing categories.

Include a visual activity distribution chart.

Include a daily activity chart showing the number of requests and grouped activities per day.

Include a recent activity list.

The dashboard must automatically refresh when new traffic is collected.

## 8. Activity Explorer

Create a dedicated Activity Explorer panel.

This feature allows users to search and query activities based on time, category, and other filters.

Implement three primary query modes.

### A. Global Activity

Display all recorded activities across the selected project.

Show:

- Total activities.
- Total requests.
- Activity categories.
- Timeline.
- Most tested endpoints.
- Activity summaries.

### B. Single Date

Allow users to select one specific date.

Example:

20 November 2026

Display every testing activity recorded on that date.

### C. Date Range

Allow users to select:

Start Date: 17 November 2026

End Date: 20 November 2026

Display all activities between those dates.

Use inclusive date filtering.

Support timezone-aware filtering.

Use the original Burp timestamps and configurable timezone settings.

### Additional Time Filters

Support:

- Today.
- Yesterday.
- Last 7 days.
- Last 30 days.
- This month.
- Previous month.
- Custom date range.
- Custom time range.

### Category Filters

Allow filtering by:

- SQL Injection.
- IDOR.
- Authentication.
- Authorization.
- XSS.
- SSRF.
- Business Logic.
- Input Validation.
- Fuzzing.
- Reconnaissance.
- Other.

Allow selecting multiple categories simultaneously.

### Additional Search Filters

Support searching by:

- Hostname.
- URL.
- Endpoint.
- HTTP method.
- Parameter name.
- Payload content.
- HTTP response status.
- Activity category.
- Confidence score.

All filters must be combinable.

Example:

Show SQL Injection and IDOR activities between 17 November and 20 November 2026, targeting api.example.com.

The results must update based on the selected filters.

## 9. Activity Timeline

Create a chronological timeline.

Group activities by date.

Example:

20 November 2026

09:12 - 09:18
SQL Injection Testing
POST /api/user/search
50 requests

10:15 - 10:48
IDOR Testing
GET /api/customer/123
23 requests

11:00 - 11:35
Authentication Testing
POST /api/auth/login
28 requests

19 November 2026

09:30 - 10:15
Business Logic Testing
POST /api/payment/checkout
17 requests

Requirements:

- Sort activities chronologically.
- Display start and end timestamps.
- Display request counts.
- Display associated endpoints.
- Display testing techniques.
- Allow expanding activity details.
- Allow clicking an activity to inspect its underlying HTTP requests.

## 10. Activity Detail Viewer

When a user selects an activity, display:

- Activity name.
- Category.
- Target hostname.
- Endpoint.
- HTTP method.
- First observed timestamp.
- Last observed timestamp.
- Total associated requests.
- Detected techniques.
- Tested parameters.
- Detection evidence.
- Confidence score.
- Associated HTTP request list.

Provide access to the original Burp request and response.

Use Burp's native HTTP message editor components where supported by the Montoya API.

The user must be able to inspect the raw HTTP request and response without leaving the extension.

## 11. Detection Confidence

Every automatically classified activity must have a confidence score.

Suggested classification:

- High: Strong evidence of a specific testing technique.
- Medium: Multiple indicators suggest a testing technique.
- Low: Weak or ambiguous indicators.

Do not present automatically detected activities as confirmed vulnerabilities.

Clearly distinguish:

- Testing Activity.
- Potential Vulnerability.
- Confirmed Finding.

The extension must never automatically claim that a vulnerability exists solely because a payload was detected.

Provide an option to manually change the category and classification status.

## 12. Daily Activity Summary

Implement a summary generator.

Example output:

Date: 20 November 2026

Summary:

During this session, testing activities focused on SQL Injection, IDOR, and Authentication.

SQL Injection testing involved Boolean-based, Error-based, and Time-based payload variations against multiple API endpoints.

IDOR testing involved manipulating object identifiers in customer-related endpoints.

Authentication testing included JWT manipulation and authorization header validation.

Total HTTP Requests: 342

Total Grouped Activities: 12

The summary must be generated from actual stored activity data.

Do not fabricate testing results or findings.

Provide an option to copy the summary to clipboard.

## 13. Export Functionality

Support the following export formats:

### Markdown
Generate readable activity documentation.

Example:

# Pentest Activity Report

## 20 November 2026

### SQL Injection Testing

- Endpoint: POST /api/user/search
- Requests: 50
- Techniques: Boolean-based, Error-based, Time-based

### IDOR Testing

- Endpoint: GET /api/customer/{id}
- Requests: 23
- Techniques: Object identifier manipulation

### Authentication Testing

- Endpoint: POST /api/auth/login
- Requests: 28
- Techniques: JWT manipulation

### CSV

Export activity records with:

- Date.
- Start time.
- End time.
- Category.
- Host.
- Endpoint.
- Method.
- Request count.
- Techniques.
- Confidence score.

### JSON

Export complete structured activity data.

Allow exporting:

- Global activities.
- Selected dates.
- Custom date ranges.
- Selected categories.

## 14. Project Management

Support multiple penetration testing projects.

Each project must have:

- Project name.
- Target hostname list.
- Description.
- Creation date.
- Associated HTTP history.
- Activity records.

Allow:

- Creating a project.
- Selecting an active project.
- Renaming a project.
- Deleting a project.
- Switching between projects.

Do not mix traffic between different projects.

Allow assigning imported or existing traffic to a project where technically possible.

## 15. Database Design

Use SQLite.

Suggested tables:

### projects
- id
- name
- description
- created_at
- updated_at

### http_requests
- id
- project_id
- burp_item_id
- timestamp
- hostname
- url
- path
- method
- request_headers
- request_body
- response_status
- response_headers
- response_body
- response_length
- content_hash

### activities
- id
- project_id
- title
- category
- hostname
- endpoint
- http_method
- start_timestamp
- end_timestamp
- request_count
- confidence_score
- classification_source
- description
- manually_verified
- created_at
- updated_at

### activity_requests
- id
- activity_id
- request_id

### detection_evidence
- id
- activity_id
- request_id
- evidence_type
- matched_pattern
- evidence_description

### detection_rules
- id
- category
- rule_name
- pattern
- enabled
- confidence_weight

Add indexes for timestamp, project_id, category, hostname, and endpoint.

Implement database migrations and transaction handling.

Use prepared statements for all database operations.

## 16. UI/UX Requirements

Create a clean, professional, modern interface.

The interface should look like a native Burp Suite extension rather than a standalone web application.

Use:

- Dark theme compatible with Burp Suite.
- Clear typography.
- Compact tables.
- Consistent spacing.
- Search fields.
- Date pickers.
- Dropdown filters.
- Timeline components.
- Expandable detail panels.
- Charts where useful.

Suggested navigation:

1. Dashboard
2. Activity Explorer
3. Timeline
4. HTTP Requests
5. Projects
6. Settings

Avoid overcrowding the interface.

The Activity Explorer must be the central feature.

Use background workers for database queries and traffic analysis.

Do not perform expensive operations on the Swing Event Dispatch Thread.

Ensure UI updates are thread-safe.

## 17. Performance and Reliability

The extension must handle large HTTP histories.

Target at least 100,000 HTTP request records without making the interface unresponsive.

Requirements:

- Incremental history synchronization.
- Batch database insertion.
- Database indexing.
- Pagination.
- Lazy loading.
- Background classification.
- Duplicate detection.
- Efficient filtering.
- Avoid storing unnecessary duplicate request data.
- Configurable response body storage.
- Graceful handling of malformed requests.

Do not block Burp's HTTP processing pipeline while analyzing traffic.

Implement error handling and extension logging.

## 18. Privacy and Storage

All collected information must remain local.

Do not transmit HTTP requests, responses, credentials, tokens, or payloads to external services.

Do not implement automatic LLM API calls.

If optional AI integration is added in the future, it must be disabled by default and require explicit configuration and user consent.

Allow the user to configure:

- Database location.
- Request body storage.
- Response body storage.
- Maximum stored body size.
- Retention policy.
- Automatic synchronization.
- Timezone.

Sensitive data must not be unnecessarily duplicated or displayed in dashboard summaries.

## 19. Settings

Implement settings for:

- Automatic HTTP history synchronization.
- Real-time traffic collection.
- Activity detection enable/disable.
- Detection category configuration.
- Detection confidence threshold.
- Timezone.
- Database path.
- Request/response body storage.
- Maximum storage size.
- Export preferences.
- Data retention.

Provide a button to manually synchronize HTTP history.

Provide a button to rebuild activity classifications from stored request data.

## 20. Important Implementation Constraints

1. Use the official Montoya API.
2. Do not use deprecated Burp Extender APIs.
3. Do not modify or replay target HTTP requests.
4. Do not send additional requests to target applications.
5. Do not treat payload detection as proof of vulnerability.
6. Preserve original timestamps whenever available.
7. Support global and date-range queries.
8. Support multiple projects.
9. Persist data between Burp Suite sessions.
10. Keep the extension fully functional without external services.
11. Do not store passwords, cookies, or authentication tokens in activity summaries unnecessarily.
12. Avoid blocking Burp Suite while processing large HTTP histories.

## 21. Development Workflow

Develop the project in stages.

### Phase 1: Foundation
- Initialize Maven project.
- Configure Montoya API.
- Register extension.
- Create the main Burp tab.
- Implement SQLite database.
- Implement project management.

### Phase 2: HTTP History Integration
- Retrieve existing HTTP history.
- Capture new traffic.
- Persist HTTP metadata.
- Implement synchronization and duplicate detection.

### Phase 3: Activity Detection
- Implement detection rules.
- Implement classification engine.
- Implement confidence scoring.
- Implement activity correlation.
- Implement manual classification.

### Phase 4: Dashboard and Explorer
- Implement dashboard.
- Implement global activity.
- Implement single-date query.
- Implement date-range query.
- Implement category filtering.
- Implement search functionality.

### Phase 5: Timeline and Details
- Implement chronological timeline.
- Implement activity grouping.
- Implement activity detail viewer.
- Integrate original Burp request/response inspection.

### Phase 6: Summary and Export
- Implement daily summaries.
- Implement Markdown export.
- Implement CSV export.
- Implement JSON export.

### Phase 7: Optimization
- Test with large HTTP histories.
- Optimize database queries.
- Test synchronization.
- Test timezone handling.
- Test persistence after Burp restart.
- Fix UI and threading issues.

## 22. Acceptance Criteria

The project is considered complete when:

1. The extension loads successfully in Burp Suite.
2. It can retrieve existing HTTP history.
3. It can capture newly generated HTTP traffic.
4. It can persist collected data in SQLite.
5. It can distinguish activities by date.
6. It can display all activities globally.
7. It can query a specific date.
8. It can query a custom date range.
9. It can filter activities by testing category.
10. It can search by hostname, endpoint, method, parameter, and payload.
11. It can group related requests into logical testing activities.
12. It can display evidence behind each classification.
13. It can display original HTTP requests and responses.
14. It can generate daily activity summaries.
15. It can export activity reports.
16. It can preserve historical records after restarting Burp Suite.
17. It can handle large HTTP histories without freezing Burp Suite.
18. It does not generate additional traffic to target systems.

## 23. Final Instructions to the Coding Agent

Before implementation:

1. Inspect the latest official Montoya API documentation.
2. Verify which methods are available for HTTP history retrieval, timestamps, HTTP handlers, and native request/response viewers.
3. Do not assume API methods exist without verifying them.
4. Design the database and architecture before implementation.
5. Implement the application incrementally.
6. Compile and test after each development phase.
7. Do not leave placeholder implementations for core features.
8. Do not create mock data as a substitute for actual Burp traffic.
9. Provide a complete Maven project with source code, build instructions, and the final extension JAR.
10. Document any API limitations and provide practical workarounds.

The final result must be a working Burp Suite extension, not a UI prototype or a static dashboard.

Prioritize accurate activity reconstruction, historical date-based querying, reliable data persistence, and a usable Activity Explorer.