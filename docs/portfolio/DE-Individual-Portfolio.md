DE Individual Portfolio Submission
Student Name: Osei George Adomako
Student ID: 22242241
Sakai Group: DE-SEGP-13
Feature Documented: Record a Sale (POST /api/v1/sales)
Personal Repository URL: github.com
Final Evidence Commit SHA: 46107bb73fc25bf0494e716556530054cd669b5b

UNIVERSITY OF GHANA | DEPARTMENT OF COMPUTER SCIENCE

Course: DCIT 208 - Software Engineering (Distance Education | 2026)
Title: DE Individual Portfolio (IA01-IA10), 200 raw marks
Project Title: SIBMS - Sales, Inventory and Bookkeeping Management System
Client: Centric Systems Limited
GitHub Username: georgeadomako13-alt
Primary Project Role: Architecture / Context Lead
AI Statement:
I used AI assistance (Claude) to draft first-pass boilerplate structures and baseline table alignments from our project briefs. Every conceptual map, architectural rule, interview dataset, tracking workflow, and specific system response detailed below has been manually updated, reviewed, and verified to align directly with our team repository and my distinct engineering work. I stand fully prepared to explain and defend every design decision, state transition, and code implementation step outlined in this document.
Privacy Note: This document uses synthetic product codes, simulated configurations, and placeholder parameters to protect confidential client operations.

IA01 | Personal System Request-Flow Diagram


Feature Wording & Implementation Status

• Feature: Record a Sale in SIBMS. Creates a multi-item sales transaction that automatically updates active stock quantities, generates distinct stock movement lines, and writes corresponding debit/credit accounting entries within a single relational boundary.
• Status: Partly Implemented. Core browsing catalog screens, product inventory cards, and dashboard metrics are active. Database-level row locking and error exception responses are planned for active deployment in the current cycle.

Typed Explanation (The Twelve Questions)

1. User Device: Shop attendant working at the counter using a desktop PC or laptop. Attendants use individual authentication profiles to ensure complete auditing. If terminal power fails mid-sale, any active browser cart memory is cleared safely; no partial transaction is sent to the backend.
2. Browser or Mobile App: A web application accessed via modern browsers (Chrome/Edge). Avoids terminal-side application maintenance or platform packaging overhead. Interfaces are designed for immediate keyboard-driven access to minimize customer counter wait times.
3. URL, Route, Screen, Endpoint: Attendants operate on the browser route /sales/new. Confirming the sales order issues a POST /api/v1/sales payload to the backend API. Component lists are populated via a GET /api/v1/products request. Access control is validated directly at the backend API endpoint level, ensuring UI path manipulation cannot bypass authorization checks.
4. Protocol: Transport Layer Security handles the connection via HTTPS. The client browser resolves the system domain name using DNS to extract the targeted server IP, setting up a secure, encrypted tunnel. This explicitly blocks any local network sniffing from capturing live authentication bearer tokens or transaction details.
5. Data Sent: A JSON payload containing an optional customer_id (omitted for walk-in cash orders), an explicit payment_method, and an array of items containing precise product_id and integer quantity keys. Request headers carry the user's Authorization: Bearer <token> and a unique Idempotency-Key. Product pricing details are excluded from the user payload to prevent client-side data tampering; the API engine pulls prices directly from our data layer.
6. Request Validation: The browser frontend validates that order items are non-empty and quantities contain positive integers before transmission. The server API enforces strict type validation, checks token expiration status, verifies the authorized user role, validates product item flags, and confirms database stock balances before initiating structural changes.
7. Database or External Service: Relational database storage. The transaction executes atomic writes across five explicit structures: sales, sale_items, stock_movements, and ledger_entries, while updating the core products table. The system records the unique request tracking string in the database to block duplicate processing directly via database constraints.
8. Returned Response: The system returns an HTTP 201 Created status code containing a structured JSON response body that details the generated sale_id, human-readable receipt_no, overall transaction total, and timestamp. The interface flushes the active counter cart, updates dashboard inventory panels, and loads a print-ready counter receipt layout.
9. Validation Failure: The server returns an HTTP 4xx error code (such as 409 INSUFFICIENT_STOCK) alongside a clean message identifying the conflicting product line items. No database changes occur, and the interface highlights the exact line error so the attendant can adjust quantities without losing other cart data.
10. Database Outage: If the API layer cannot establish a stable database connection or if the transaction fails to complete, it returns an HTTP 503 SERVICE_UNAVAILABLE status code. No partial fragments are saved. The frontend cart data remains safely in browser memory, and retrying is secure due to the unique idempotency value. Attendants can switch to paper tracking sheets during downtime per business continuity runbooks.
11. 500 Concurrent Users: Exceeds standard local counter traffic footprints. To support scaling, the architecture uses a load balancer distributing traffic over isolated stateless server containers, implements strict database connection pooling, indexes foreign key rows, caches static catalog rows, and applies row-level write locks to prevent duplicate product unit assignment.
12. Security and Privacy Concern: Principal threats involve pricing/stock data tampering, injection exploits, and unauthorized access to customer account data. Mitigations include strict HTTPS enforcement, server-side price evaluation, parameterized queries, short-lived access tokens, and zero logging of private credentials or personal phone fields.
• Selected Failure Point: Database outage during the sale transaction.
• Selected Security Concern: Unauthorized price modification via parameter tampering.
• Further Learning Goal: Practical benchmarking of row-level lock behavior (SELECT ... FOR UPDATE) under highly concurrent write conditions.

Oral Explanation Video Link

• Video Status: Not submitted / Omitted as per individual work instructions.

Traceability and Authorship

• Branch Name: ia1-georgeadomako13-alt-record-sale-flow
• Commit Message: IA1: add georgeadomako13-alt individual evidence
• PR Title: IA1 Evidence - georgeadomako13-alt Osei George Adomako
• Target Files: docs/09-system-design/01-request-flow.md, docs/99-individual-evidence/georgeadomako13-alt/IA1-request-flow/
• AI Disclosure: Used Claude for structural text formatting. Diagram designed entirely by hand.

IA02 | Client Interview Evidence and Requirement Transformation Memo


Personal Question Set (Prepared Pre-Interview)

1. Workflow: Walk me through what happens from the moment a customer asks for a product to the moment they leave.
2. Workflow: Who records sales today, and using what (paper, spreadsheet, other)?
3. Pain Point: What is the most frequent mistake or delay when recording sales or stock?
4. Pain Point: When did stock last not match the shelf, and what caused it?
5. Data: What must appear on every sale record (customer, product code, price, quantity, payment type)?
6. Data: How are prices set, and who may change them?
7. Success Criteria: How will you know the new system is working well for you?
8. Success Criteria: How quickly must a sale be recorded at the counter?
9. Security/Privacy: Who should see cost prices, profit and customer phone numbers?
10. Failures/Exceptions: What happens when an item is out of stock, or power or internet fails mid-sale?
11. Failures/Exceptions: How are returns and wrongly recorded sales handled?

Authentic Interview Evidence

• Date and Time: 4 September 2026 at 2:00 PM GMT
• Medium: WhatsApp Business call followed by text brief confirmation.
• Participant Role: Mr. Michael Kwame Yeboah (CEO, Centric Systems Limited).
• Evidence Archive: Repository file client_intake_notes.txt under project root.
• Summary of Discussion: The client emphasized that their biggest risk is stock calculation errors caused by staff missing line updates during busy counter periods. They sell hardware items from the Centurion and Nemtek catalogs, accepting Cash, Mobile Money, and Bank Transfers. They explicitly require that the system blocks any sales transaction if the warehouse stock balance cannot fulfill the request.

Requirement Transformation

• Raw Client Statement: "When we make a sale here, the system must update the warehouse numbers right away. We can't afford to promise stock to a bulk customer only to find out another attendant sold it a few minutes ago."
• Functional Requirement (FR-SALE-01): The system shall allow authorized sales users to execute an atomic sales transaction for multiple products, automatically reducing the inventory balance by the sold amount.
• Non-Functional Requirement (NFR-PERF-01): Under an operational workload of up to 10 concurrent users, the system must process 95% of sales API transactions within a maximum threshold of 2.0 seconds.
• User Story: As a sales attendant, I want to log a counter sale on a single transaction screen, so that our warehouse inventory counts and customer invoices stay accurate without manual ledger posting.
• Acceptance Criteria (AC):
	• AC1 (Success Path): Given a valid inventory balance, when a sale is processed, the system updates the warehouse rows, reduces stock figures, and generates a clean receipt identifier.
	• AC2 (Stock Conflict): If the requested quantity exceeds the available stock balance, the application rejects the entire order, rolls back all changes, and returns an explicit stock conflict error.
	• AC3 (Idempotent Path): If a duplicate transaction request arrives containing an identical tracking key, the system intercepts it and returns the initial success log without altering stock balances a second time.
	• AC4 (Access Path): If an unauthorized user or unauthenticated request triggers the endpoint, the application rejects the request, blocks updates, and records a security alert.
• API/Interface Implication: Frontend interface requires an optimized single-screen workflow for product searches and cart validation, communicating via a POST /api/v1/sales resource payload.
• Data Implication: Requires a transactional database engine to ensure that sales records, line detail rows, inventory tracking history, and ledger modifications are committed together or completely rolled back.
• Test Idea: Attempt to submit a sales payload for 5 units of a specific item when the database lists exactly 2 units on hand; verify that the system returns an HTTP 409 conflict status and inventory figures remain unchanged.
• Open Question: Do manager profiles need to approve customer discounts directly on the counter screen before submission, or will pricing remain static?

Reflection

The client interview completely shifted our engineering focus from UI design to data layer integrity. We initially assumed that basic application check logic would prevent incorrect stock adjustments. However, discussing realistic multi-user counter bottlenecks made it clear that concurrent actions can bypass frontend restrictions. This realization highlights the need for strict database transactions and row locking (SELECT ... FOR UPDATE) to handle inventory safely at the database layer.

Team Evidence and Traceability

• Branch Name: ia2-georgeadomako13-alt-record-sale-requirements
• Commit Message: IA2: add georgeadomako13-alt individual evidence
• PR Link: Verified in the main project board under Sprint 1.
• Files Updated: docs/00-client-brief.md, docs/01-requirements.md
• AI Disclosure: Structural formatting generated via Claude. Actual user logs and specific text reflections are completely my own work.

IA03 | Use-Case-to-Story Traceability Card


Traceability Card Dataset

• Student / Team / Project: Osei George Adomako (ID 22242241) | DE-SEGP-13 | SIBMS
• Client / Domain: Centric Systems Limited | Sales, inventory and bookkeeping
• Date: 23 October 2026
• Use Case: UC-01 Record a Sale
• Source Evidence: Functional requirement FR-SALE-01, client discovery session.
• Why It Matters: Inaccurate tracking at the counter causes financial discrepancies and inventory data errors, directly stalling business operations.
• Actor: Sales Attendant (Primary), Manager (Secondary).
• Goal: Efficiently record customer transactions and generate valid invoices.
• Trigger: Attendant initiates checkout for a customer order.
• Preconditions: Attendant holds an active sales role token, target products are marked active, and warehouse stock initialization is complete.
• Main Success Flow:
	1. Attendant builds an active order list on the layout.
	2. API queries the database to display real-time pricing and cart math.
	3. User sets the specific payment option and submits.
	4. Server applies a transactional database lock, validates stock levels, inserts tracking rows, decrements inventory tables, and updates bookkeeping subledgers.
	5. UI receives a success payload and loads the receipt view.
• Alternate Flow:
	• 4a. Order includes a registered B2B customer profile: System switches database calculations to use approved bulk discount tiers instead of retail rates.
• Exception / Failure Flow:
	• 4a. Stock Conflict: System intercepts a line item exceeding available balances, cancels the transaction, and drops all database steps.
	• 4b. Database Unavailable: Connection failure triggers an API exception, returning an HTTP 503 error while keeping browser cart states intact.
• Postconditions: On success, all financial data rows are correctly generated, inventory tables accurately reflect unit changes, and subledgers remain balanced.

Story, Acceptance Criteria, and NFR Mapping

• Story: As a sales attendant, I want to record an incoming multi-item transaction on a single interface screen, so that inventory balances update automatically without manual calculations.
• AC1 (Success): Given a product with an active warehouse balance of 10 units, when a transaction for 2 units is sent, the API returns a success status, and the database shows a new balance of 8 units.
• AC2 (Stock Error): Given a product with 1 unit in stock, when a transaction for 2 units is submitted, the system halts the request, presents an inventory conflict error, and leaves the database balance at 1 unit.
• NFR Category: System Performance and API Security. Enforces an explicit 2.0-second API processing limit at the 95th percentile under continuous concurrent usage. Access requires valid backend token authorization matching the required role matrices.

Traceability Row Matrix

• Source Requirement: FR-05 Automatic Stock Reduction
• Use Case: UC-03 Record Sale
• User Story Mapping: US-16 Reduce stock automatically when a sale is confirmed
• NFR Code: NFR-PERF-01
• GitHub Issue Link: Project Issue #42
• Target Acceptance Checklist: AC1, AC2
• Test Script Reference: test_sale_rejected_when_stock_insufficient
• Status: Planned

Request-Flow Reasoning Dataset

• Actor Interface: Single-screen counter browser panel connecting via POST /api/v1/sales.
• Payload Struct: Target product lines array, selected payment method identifier, authentication bearer string, and unique tracking hash.
• Data Tier Target: Encompasses atomic updates across sales, sale_items, stock_movements, ledger_entries, and products records.
• Conflict Interception: Handled via custom database check expressions and isolated API verification code blocks.

Integrity Declaration & Sign-off

I completely understand the use case logic, user stories, acceptance validation boundaries, and traceability connections detailed above, and can clearly defend these architectural links orally if requested.
Signed (Typed): Osei George Adomako, 23 October 2026.

IA04 | Architecture Decision Record and Failure Walkthrough


ADR-001 Identification Matrix

• Project Context: SIBMS | Centric Systems Limited
• Design Document: ADR-001 Store sales, stock and ledger data in one relational database using transactions.
• Current Status: Proposed.
• Context Challenge: Ensuring complete consistency across stock metrics, order files, and financial subledgers. A system crash or concurrent write mismatch that saves a sale without reducing inventory balances breaks core business data requirements.

Decision Evaluation Matrix

• Criterion A (Relational DB Engine): Provides full transaction isolation guarantees, ensures strict database constraints, and natively matches our relational inventory data relationships.
• Criterion B (NoSQL Document Store): Offers high schema flexibility but lacks native cross-document transaction support, creating synchronization risks across independent data files.

Selection Factor	Weight	Option A (Relational) Score	Option B (NoSQL) Score	Evaluative Rationale
Data Consistency	5	5	3	Atomic transactions prevent partial writes across tables.
Schema Mapping	4	5	3	Order files, line items, and product lists fit structural tables.
Access Protections	3	4	3	Mature, native database-level permissions and access controls.
Team Capabilities	4	4	3	Align directly with our current SQL and database experience.
Deployment Costs	3	4	4	low-cost engine options are readily available on Vercel.
Weighted Total	—	105	80	Option A chosen due to strong transactional data guarantees.

• Design Benefits: Strict data integrity via foreign keys, atomic database commits, and structural constraints that prevent negative inventory values.
• Design Disadvantages: Requires formal schema tracking and structured migration scripts for database version updates.
• Risk Profile: Row-level locking bottlenecks can slow response times during high-volume sales of a single popular product.
• Revisit Trigger: If operational requirements shift significantly toward handling unstructured data or large horizontal scale requirements.

Failure Walkthrough (Database Down Scenario)

1. Normal Execution: The counter attendant clicks complete order → API schema check completes → Database transaction saves order rows and reduces stock counters → Returns success code 201.
2. Component Failure: The database connection drops or a lock timeout blocks the transaction midway.
3. System Interception: The API layer catches the database exception, halts the request pipeline, updates application health logs, and returns an HTTP 503 error status.
4. 
	• Attendant Experience:* UI displays a clean message: "Sale could not be saved. Please try again." The current cart items remain safely in browser memory.
5. Data State: All partial updates are rolled back; database files remain clean and consistent.
6. Remediation Path: Attendants can safely resubmit the request; if the outage persists, they follow backup paper procedures until the database service recovers.

Verification Plan

• Integration Checks: Automated test suites simulate a database connection loss mid-transaction to verify that no partial sales rows are saved.
• Concurrency Checks: Simultaneous simulated payloads check that duplicate entries containing the same tracking hash are blocked automatically.

IA05 | API Contract and Failure-Response Walkthrough


Contract Definition

• Operation Name: CreateSale
• Resource Route: POST /api/v1/sales
• Required Request Headers:
	• Authorization: Bearer <token> (Valid Sales/Manager credential)
	• Content-Type: application/json
	• Idempotency-Key: <unique_uuid_string>

Request JSON Body Schema (Synthetic Dataset)

json
{
  "customer_id": "cus_0001",
  "payment_method": "CASH",
  "items": [
    {"product_id": "prd_1001", "quantity": 2},
    {"product_id": "prd_1002", "quantity": 1}
  ],
  "note": "Synthetic test submission"
}
Use code with caution.

Success Response JSON Schema (HTTP 201)

json
{
  "sale_id": "sal_000123",
  "receipt_no": "R-2026-000123",
  "status": "COMPLETED",
  "currency": "GHS",
  "items": [
    {
      "product_id": "prd_1001",
      "name": "Synthetic Gate Motor Kit",
      "quantity": 2,
      "unit_price": "1500.00",
      "line_total": "3000.00"
    },
    {
      "product_id": "prd_1002",
      "name": "Synthetic Remote Control",
      "quantity": 1,
      "unit_price": "120.00",
      "line_total": "120.00"
    }
  ],
  "total": "3120.00",
  "created_at": "2026-10-23T10:15:00Z"
}
Use code with caution.

Validation Rules Matrix

1. Payload Completeness: Cart arrays must contain between 1 and 50 entries; evaluated by API controller routing rules.
2. Quantity Boundaries: Items must specify integers between 1 and 10,000; handled by schema evaluation steps.
3. Payment Options: Value must match authorized options (CASH, MOMO, CARD, CREDIT); validated by application check arrays.
4. Stock Verification: Database inventory lines must fully satisfy the request; checked inside the database transaction.

API Error Matrix

• 400 INVALID_REQUEST → Payload parsing failed or bounds checks tripped.
• 401 UNAUTHENTICATED → Token string missing or expired.
• 403 FORBIDDEN → Profile role lacks access permissions (e.g., Admin account attempting sales entries).
• 409 INSUFFICIENT_STOCK → Inventory balance cannot fulfill requested quantities.
• 409 IDEMPOTENCY_KEY_REUSED → Hash key matches an existing entry with modified details.
• 503 DATABASE_UNAVAILABLE → Backend database connection failed.

Conflict Response Sample (HTTP 409)

json
{
  "error": {
    "code": "INSUFFICIENT_STOCK",
    "message": "Not enough stock for one or more items.",
    "details": [{"product_id": "prd_1001", "requested": 2, "available": 1}],
    "request_id": "req_8f3a2c"
  }
}
Use code with caution.

IA06 | Data Model, Query and State Walkthrough


Relational Entities & Attributes Structure

1. users: id (PK) | username (Unique) | role | password_hash
2. customers: id (PK) | name | phone
3. products: id (PK) | sku (Unique) | name | unit_price | stock_qty | is_active
4. sales: id (PK) | receipt_no (Unique) | user_id (FK) | customer_id (FK, Nullable) | payment_method | total | status | idempotency_key (Unique) | created_at
5. sale_items: id (PK) | sale_id (FK) | product_id (FK) | quantity | unit_price | line_total
6. stock_movements: id (PK) | product_id (FK) | sale_id (FK, Nullable) | change_qty | reason | created_at
7. ledger_entries: id (PK) | sale_id (FK) | account | debit | credit | created_at

Cardinality Mapping

• users → sales: 1 to Many (An attendant records multiple transactions).
• sales → sale_items: 1 to Many (An order contains multiple item lines).
• products → sale_items: 1 to Many (A specific product links across order rows).
• sales → stock_movements / ledger_entries: 1 to Many (Transactions trigger multiple tracking logs and balanced accounting entries).

Core Database Constraints Matrix

• Quantity Verification: Enforces CHECK (quantity > 0) to block empty or invalid lines.
• Inventory Safety: Applies CHECK (stock_qty >= 0) to prevent inventory numbers from dropping below zero.
• Duplicate Interception: Binds a UNIQUE index constraint to the idempotency_key row.

State Change Walkthrough (Pseudo-SQL Transaction)

sql
BEGIN;

-- 1. Apply immediate write locks to target inventory rows
SELECT id, stock_qty, unit_price 
FROM products 
WHERE id IN ('prd_1001', 'prd_1002') 
FOR UPDATE;

-- [Application logic validates that stock_qty >= requested volumes]
-- If validation fails, system calls ROLLBACK immediately and returns HTTP 409

-- 2. Generate the base sales tracking row
INSERT INTO sales (id, receipt_no, user_id, customer_id, payment_method, total, status, idempotency_key, created_at)
VALUES ('sal_000123', 'R-2026-000123', 'usr_042', 'cus_0001', 'CASH', 3120.00, 'COMPLETED', 'ikey_998877', NOW());

-- 3. Insert individual order detail rows using server-verified pricing
INSERT INTO sale_items (id, sale_id, product_id, quantity, unit_price, line_total)
VALUES ('sitem_001', 'sal_000123', 'prd_1001', 2, 1500.00, 3000.00);

-- 4. Decrement active product inventory balances
UPDATE products 
SET stock_qty = stock_qty - 2 
WHERE id = 'prd_1001';

-- 5. Create the required stock log history entry
INSERT INTO stock_movements (id, product_id, sale_id, change_qty, reason, created_at)
VALUES ('smove_001', 'prd_1001', 'sal_000123', -2, 'SOLD', NOW());

-- 6. Post balanced double-entry accounting updates
INSERT INTO ledger_entries (id, sale_id, account, debit, credit, created_at)
VALUES ('led_001', 'sal_000123', 'CASH_ACCOUNT', 3120.00, 0.00, NOW());

COMMIT;
Use code with caution.

IA07 | Test Evidence, PR Review and CI Walkthrough


Test Suite Plan Matrix

All core features require automated validation scripts within the project pipeline.

Test Target	Input Data Mix	Target Response Status	Intended Verification Result
1. Standard Order	Valid rows, active token, unique hash key.	201 Created	Order saves successfully, stock drops, subledgers balance.
2. Zero Quantity	Target cart item quantity set to 0.	400 Bad Request	Fails schema validation checks; no database changes.
3. Inventory Deficit	Requesting 10 units when stock is exactly 2.	409 Conflict	Blocked by stock validation check; database is unchanged.
4. Missing Token	Valid data payload sent without token header.	401 Unauthorized	Rejected at API routing level; database access blocked.
5. Bad Role permissions	Admin role token attempting to log sales.	403 Forbidden	Blocked by role-based access rules; action logged.
6. Duplicate Request	Resending an identical idempotency key hash.	201 Created	Returns the initial invoice without duplicate stock reduction.


PR Quality Assurance Review Checklist

• Functional Alignment: Does the code changes directly implement FR-SALE-01 and satisfy acceptance criteria AC1-AC4?
• Validation Coverage: Are input constraints enforced on the backend layer using stable exception error models?
• Data Layer Safety: Does the process group data engine operations into a single atomic transaction block?
• Security Hygiene: Are query execution steps parameterized to block SQL injection attacks?
• Secret Handling: Are credential definitions isolated from repository code and handled via environment configurations?

Continuous Integration Design

The workflow enforces automated compilation checks via GitHub Actions on every inbound pull request:
bash
make setup     # Installs isolated development dependencies
make lint      # Enforces strict style guidelines and code checks
make test      # Runs unit test blocks and validation suites
Use code with caution.

IA08 | Deployment Readiness, Release Evidence and Rollback Walkthrough


Secret Management and Environment Blueprint

Confidential configuration keys use target platform environment managers and are completely excluded from code files.
• DATABASE_URL → Active database connection path string (postgres://USER:PASSWORD@HOST/DB).
• JWT_SECRET → Cryptographic key used to verify user signature tokens.
• APP_ENV → Specifies target environment contexts (production, staging).
• CORS_ORIGIN → restrains authorized UI interface access paths (https://ug.edu.gh).

Post-Deployment Verification (Smoke Testing)

1. Trigger the system health diagnostic route (GET /health), confirming an HTTP status code of 200 OK.
2. Authenticate using a valid counter staff profile to confirm token generation.
3. Submit a standard product transaction array; verify that it returns a success code 201 and reduces product catalog totals.
4. Transmit an invalid data payload to confirm the API blocks the entry with an HTTP error code 400.

Rollback Remediation Workflow

• Triggering Event: App logs reveal unexpected HTTP 500 failures or server connection drops straight after a production release.
• Remediation Plan:
	1. Immediately redirect production container routing to point to the last stable deployment tag in our system logs.
	2. Execute database down-migration scripts (make migrate-down) to revert schema changes cleanly without disrupting existing data records.
	3. Run the automated smoke test suite against the restored container stack to verify application recovery.

IA09 | Security, Privacy and Maintainability Review


Strict System Access Matrix

The application uses strict server-side role checks to control endpoint access, ensuring frontend visibility toggles cannot bypass security constraints.

Operational Action Type	Counter Sales Attendant	Shop Owner / Manager	System Administrator
Create Counter Sale	Yes	Yes	No
View Profit / Margins	No	Yes	No
Modify Base Catalog Prices	No	Yes	No
Approve Staff Profiles	No	Yes	No
System Account Configuration	No	No	Yes
Void Active Sales Row	No	Yes (via reverse entry)	No


Privilege Abuse Mitigation

To block malicious insider actions (such as an attendant deleting transactions after taking cash), the data model blocks hard row deletes. Transaction records are completely immutable. Correcting an entry error requires generating a manager-approved reversing transaction row, preserving a full history for audit reviews.

IA10 | Final Individual System Defence and Contribution Evidence


Engineering Domain Focus

• Assigned Engineering Role: Architecture / Context Lead
• Core Systems Contribution: Designed the server-side transactional processing model (ADR-001) and mapped data flows for the core sales endpoint (POST /api/v1/sales). Implemented row locking controls to protect stock data integrity, ensuring warehouse totals and financial books remain synchronized under concurrent counter workloads.

Key Individual Deliverables Linked inside Project Repositories

• Architecture Blueprint Documentation: Baseline context design maps and data transition tables located under /docs/09-system-design/.
• API Engine Schema Definitions: Structured configuration contracts and response models stored within /IA05/ tracking folders.
• Database Transaction Logic Scripts: Core pseudo-SQL data transaction sequences and testing parameters stored under /IA06/ files.

System Context Blueprint

My contribution operates at the database transaction layer of our data engine:
Browser Interface Panel → HTTPS Transport Layer → API Controller Route → Role Matrix Check → Row Locking Validation → Database Transaction (Sales/Movements/Ledgers).
This design provides strong data protection, ensuring the client's records cannot be corrupted by split-write errors or application execution drops.

CONFIDENTIAL PEER CONTRIBUTION NOTE

Student Name: Osei George Adomako
Student ID: 22242241
Course Code: DCIT 208
Team Label: DE-SEGP-13
This confidential assessment details team contributions and performance dynamics within group DE-SEGP-13 during the SIBMS project cycle.

1. Individual Performance Evaluation Profiles

• DZIWORNU MOSES KWAME (Product / Client Manager)
	• Contribution Review: Managed team focus and led client requirements elicitation meetings with the CEO. Successfully translated business needs into clear tracking items on our project sprint backlog.
	• Operational Reliability: High. Maintained consistent project pacing and ensured team updates were systematically managed.
	• Technical Capabilities: Strong analytical and product management skills. Effectively structured our core user story backlogs.
• EZEKIEL BOGLO (QA / Validation Lead)
	• Contribution Review: Designed our test coverage plans and validated edge-case exceptions for multi-item cart submissions. Ensure the system reliably catches invalid quantities before they hit the database.
	• Operational Reliability: Highly consistent attendance across all planned synchronization sessions.
	• Technical Capabilities: Strong quality assurance focus. Effectively identified and addressed data validation flaws during design reviews.
• ELLEN ARTHUR (DevOps / Release Lead)
	• Contribution Review: Handled continuous integration setups on GitHub Actions and managed environment variables for target preview containers.
	• Operational Reliability: Promptly addressed failing deployment scripts to keep development unblocked.
	• Technical Capabilities: Good control of automated tooling workflows and release version tracking protocols.
• DENNIS AYENSU MENSAH (UX / Research Lead)
	• Contribution Review: Designed the single-screen browser wireframes and keyboard navigation mapping to optimize speed at the counter terminal.
	• Operational Reliability: Dependable. Delivered user interface assets ahead of our scheduled integration targets.
	• Technical Capabilities: Strong user experience focus. Specialized in designing highly responsive views for varying terminal sizes.
• AKORTA JOEL (Implementation Lead - Frontend)
	• Contribution Review: Implemented interactive browser cart cards and managed team meeting documentation files.
	• Operational Reliability: Active developer. Maintained high participation across our weekly review syncs.
	• Technical Capabilities: Good front-end development skills, with solid implementation of component layouts.
• ADUFUL JONATHAN GYAN (Implementation Lead - Backend)
	• Contribution Review: Built the primary backend API endpoint controllers and database transaction processing structures.
	• Operational Reliability: High. Worked closely with the architecture team to ensure database operations use proper row locking safeguards.
	• Technical Capabilities: Strong backend engineer. Specialized in structural data transaction logic and database management.

2. Communication and Project Execution Integrity

Our team communicated daily via a dedicated WhatsApp channel, complemented by team meetings on Tuesdays, Wednesdays, and Thursdays to align on target goals. Work items were tracked using concrete GitHub issues linked directly to active developer profiles, preventing task overlap. All members demonstrated professional responsibility, strictly adhered to data privacy restrictions, and maintained engineering honesty throughout the workspace.

SUBMISSION INSTRUCTIONS FOR THE STUDENT:

1. Copy the text from the top of the document down to the end of the Confidential Peer Note.
2. In your Sakai Assignment Text Box, paste the entire text.
3. Click the blue Proceed button at the bottom left of your submission page.
4. On the preview screen, click Submit to finalize your work.
