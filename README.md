# Vultrack - Pentest Management System (PMS)

A light-weight, high-performance Penetration Testing Management System built with the **Grit Go Framework**. Designed for security auditors, red teams, and DevSecOps practitioners to log findings in real-time, track remediation lifecycles, and streamline retesting workflows without the overhead of heavy enterprise tools.

---

## 🌟 Key Features

* **Real-time Finding Logging**: Streamline vulnerability submission during active engagements with Markdown/PoC support, dynamic CVSS v3.1/v4.0 score calculation, and structured request/response artifacts.
* **Engagement & Scope Boundaries**: Organize assessments by project, environment (External, Internal, Web, Mobile, Cloud), target IP/FQDN range, and assigned auditor team.
* **Remediation Lifecycle Engine**: Full status state-machine (`Draft` ➔ `Confirmed` ➔ `Assigned` ➔ `Pending Retest` ➔ `Resolved` / `Risk Accepted`).
* **Tool Result Ingestion**: Native parser pipeline for common pentest tools (`Burp Suite XML`, `Nmap XML`, `Nuclei JSON`, `Nessus`).
* **Jira / GitHub Issue Sync**: One-click escalation of confirmed vulnerabilities into developer issue trackers with full back-link context.
* **Automated Executive Reporting**: Export clean, structured Markdown, PDF, or HTML executive summaries and detailed technical findings.

---

## 🏗️ Architecture & Tech Stack

* **Framework**: [Grit Go Framework](https://www.google.com/search?q=https://github.com/grit-framework) (High-performance, modular Go REST/gRPC backend)
* **Database**: PostgreSQL / SQLite (GORM with automated migrations)
* **Auth**: JWT / OAuth2 / RBAC (Roles: `Admin`, `Auditor`, `Client / Developer`, `Read-Only Observer`)
* **Frontend Integration**: Grit REST API / Inertia-friendly endpoints / OpenAPI 3.0 / Swagger UI

---

## 📂 Repository Structure

```text
.
├── cmd/
│   └── server/          # Main application entry point
├── config/              # Environment & system configurations
├── internal/
│   ├── audit/           # Audit logs & compliance history
│   ├── engagement/      # Engagement & scope domain module
│   ├── finding/         # Vulnerability & CVSS domain module
│   ├── parser/          # Tool import parsers (Burp, Nuclei, Nmap, Nessus)
│   ├── retest/          # Retesting verification workflow module
│   └── user/            # Auth, RBAC & identity module
├── migrations/          # SQL migration files
├── pkg/
│   ├── cvss/            # CVSS v3.1 / v4.0 calculation engine
│   └── exporter/        # PDF / Markdown / HTML report generation
├── .env.example
├── docker-compose.yml
├── Makefile
└── README.md

```

---

## 🗄️ Database Entity Overview

```text
[ Engagement ] 1 ─── * [ Finding ] 1 ─── * [ FindingHistory / Retest ]
      │                      │
      ├── Target Scopes      ├── Evidence / Attachments (PoC)
      └── Team Members       └── External Issue Links (Jira/GitHub)

```

### Core Data Schema

#### 1. Engagements (`engagements`)

* `id` (UUID, Primary Key)
* `title` (VARCHAR)
* `client_name` (VARCHAR)
* `start_date`, `end_date` (TIMESTAMP)
* `status` (`planning`, `active`, `reporting`, `completed`, `archived`)
* `scope_definition` (TEXT / JSONB)

#### 2. Target Scopes (`target_scopes`)

* `id` (UUID)
* `engagement_id` (UUID, FK)
* `target` (VARCHAR - e.g., `192.168.1.0/24`, `api.example.com`)
* `type` (`ip`, `cidr`, `domain`, `url`, `repo`)

#### 3. Findings (`findings`)

* `id` (UUID, Primary Key)
* `engagement_id` (UUID, FK)
* `title` (VARCHAR)
* `cwe_id` (INT - e.g., `89` for SQLi)
* `severity` (`critical`, `high`, `medium`, `low`, `info`)
* `cvss_score` (NUMERIC)
* `cvss_vector` (VARCHAR)
* `description` (TEXT)
* `proof_of_concept` (TEXT - Markdown with raw requests/responses)
* `remediation` (TEXT)
* `status` (`draft`, `confirmed`, `assigned`, `in_fix`, `pending_retest`, `resolved`, `wont_fix`)
* `reporter_id` (UUID, FK)

---

## 🚀 Quick Start

### Prerequisites

* Go 1.22+
* PostgreSQL 15+ (or SQLite for quick evaluation)
* Make & Docker / Docker Compose

### Environment Setup

1. **Clone the repository**:
```bash
git clone https://github.com/your-org/grit-pentest.git
cd grit-pentest

```


2. **Configure environment variables**:
```bash
cp .env.example .env

```


3. **Start local database with Docker Compose**:
```bash
docker-compose up -d postgres

```


4. **Run database migrations**:
```bash
make migrate-up

```


5. **Start the Grit development server**:
```bash
make run

```


The server will bind to `http://localhost:8080`. API Documentation is accessible at `http://localhost:8080/swagger/index.html`.

---

## 🛠️ REST API Blueprint

| HTTP Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/api/v1/auth/login` | Authenticate user & obtain JWT token |
| `GET` | `/api/v1/engagements` | List all active pentest engagements |
| `POST` | `/api/v1/engagements` | Create a new engagement |
| `GET` | `/api/v1/engagements/:id/findings` | Fetch all vulnerabilities under an engagement |
| `POST` | `/api/v1/engagements/:id/findings` | Log a new finding in real-time |
| `PATCH` | `/api/v1/findings/:id/status` | Update finding lifecycle state (`pending_retest`, `resolved`) |
| `POST` | `/api/v1/findings/:id/retest` | Submit a retest verdict with updated evidence |
| `POST` | `/api/v1/import/burp` | Import scanner output directly into findings |

---

## 📄 Exporting Reports

Run the built-in CLI command to export executive reports:

```bash
# Export Markdown report for engagement UUID
go run cmd/server/main.go export --engagement-id=<UUID> --format=md --output=report.md

# Export PDF report
go run cmd/server/main.go export --engagement-id=<UUID> --format=pdf --output=report.pdf

```

---

## 🤝 Contributing

Contributions are welcome! Please follow these steps:

1. Fork the project repository.
2. Create your feature branch (`git checkout -b feature/nuclei-parser`).
3. Commit your code (`git commit -m 'Add Nuclei JSON scan output parser'`).
4. Push to the branch (`git push origin feature/nuclei-parser`).
5. Open a Pull Request.

---

## 📜 License

Distributed under the MIT License. See `LICENSE` for more information.
