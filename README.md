# 🔐 REF_PasswordReset — Automated Password Reset Passcode Generator

> **UiPath REFramework (Dispatcher + Performer)** automation that streamlines employee password reset by reading email requests, generating passcodes via a web portal, and delivering results via email.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [How It Works](#how-it-works)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Configuration](#configuration)
- [Setup & Installation](#setup--installation)
- [Test Cases](#test-cases)
- [Dependencies](#dependencies)
- [License](#license)

---

## 🧩 Overview

**REF_PasswordReset** is a production-ready UiPath automation built on the **Robotic Enterprise Framework (REFramework)**, following the **Dispatcher-Performer** pattern. It automates the end-to-end process of generating password reset passcodes for employees based on incoming email requests.

### Key Features

- 📧 **Email-triggered input** — Reads incoming IMAP emails matching a configured subject and downloads Excel attachments
- ✅ **Data validation** — Validates required fields (Username, EmpCode, Email) before processing
- 🔄 **Queue-based processing** — Uses UiPath Orchestrator queues to manage transaction items
- 🌐 **Web portal automation** — Automates passcode generation on a web application using UI Automation (Computer Vision + Fuzzy Selectors)
- 📤 **Email output delivery** — Sends generated passcodes via SMTP to the configured recipient
- 📊 **Consolidated reporting** — Generates an `AllReport.xlsx` file summarizing all processed records
- 🛡️ **Enterprise-grade resilience** — Built-in retry logic, exception handling, screenshot capture, and recovery mechanisms

---

## 🏗️ Architecture

This project follows the **Dispatcher-Performer** pattern within the REFramework:

```
┌─────────────────────────────────────────────────────────┐
│                     DISPATCHER                          │
│                                                         │
│  1. Read emails (IMAP)                                  │
│  2. Download Excel attachment                           │
│  3. Validate input data (UName, EmpCode, Email)         │
│  4. Filter valid records                                │
│  5. Push items to Orchestrator Queue                    │
└─────────────────────┬───────────────────────────────────┘
                      │
              Orchestrator Queue
                      │
┌─────────────────────▼───────────────────────────────────┐
│                     PERFORMER                           │
│                                                         │
│  1. Fetch transaction from Queue                        │
│  2. Validate transaction data                           │
│  3. Open web portal (Edge browser)                      │
│  4. Enter Username, EmpCode, Email                      │
│  5. Click "Generate Passcode"                           │
│  6. Extract generated passcode                          │
│  7. Email passcode to recipient (SMTP)                  │
│  8. Log results to AllReport.xlsx                       │
└─────────────────────────────────────────────────────────┘
```

### State Machine Flow

```
┌──────────────┐     ┌───────────────────┐     ┌──────────────────────┐     ┌─────────────┐
│              │     │                   │     │                      │     │             │
│ Initialization├────►│ Get Transaction  ├────►│ Process Transaction  ├────►│ End Process │
│              │     │     Data          │     │                      │     │             │
└──────────────┘     └───────┬───────────┘     └──────────┬───────────┘     └─────────────┘
       ▲                     │                            │
       │                     │ No more transactions       │ System Exception
       │                     └────────────────────────────┘
       └──────────────────────────────────────────────────┘
```

---

## ⚙️ How It Works

### 1. INITIALIZE PROCESS

| Workflow | Description |
|---|---|
| `Framework/InitAllSettings.xaml` | Loads configuration from `Data/Config.xlsx` (Settings & Constants sheets) and Orchestrator assets |
| `Framework/InitAllApplications.xaml` | Opens and logs into applications used in the process |
| `Dispatcher.xaml` | Reads emails, downloads input Excel, validates & filters data, pushes to Orchestrator Queue |
| `Framework/KillAllProcesses.xaml` | Kills residual application processes for a clean start |

### 2. GET TRANSACTION DATA

| Workflow | Description |
|---|---|
| `Framework/GetTransactionData.xaml` | Fetches the next queue item from the Orchestrator queue defined in `Config("OrchestratorQueueName")` |

### 3. PROCESS TRANSACTION

| Workflow | Description |
|---|---|
| `Framework/Process.xaml` | Validates transaction fields → Opens web portal in Edge → Types Username, EmpCode, Email → Clicks "Generate Passcode" → Extracts output passcode → Sends email with passcode via SMTP → Logs result to DataTable |
| `Framework/SetTransactionStatus.xaml` | Updates the queue item status: **Success**, **Business Rule Exception**, or **System Exception** |

### 4. END PROCESS

| Workflow | Description |
|---|---|
| `Framework/CloseAllApplications.xaml` | Logs out and closes all applications |

---

## 📁 Project Structure

```
REF_PasswordReset/
├── Main.xaml                        # Entry point — REFramework State Machine
├── Dispatcher.xaml                  # Reads emails, validates data, pushes to queue
├── Framework/
│   ├── InitAllSettings.xaml         # Loads Config.xlsx settings & assets
│   ├── InitAllApplications.xaml     # Opens required applications
│   ├── GetTransactionData.xaml      # Fetches next queue item
│   ├── Process.xaml                 # Core business logic (passcode generation)
│   ├── SetTransactionStatus.xaml    # Updates transaction status
│   ├── CloseAllApplications.xaml    # Closes applications gracefully
│   ├── KillAllProcesses.xaml        # Force-kills application processes
│   ├── RetryCurrentTransaction.xaml # Handles transaction retry logic
│   └── TakeScreenshot.xaml          # Captures screenshots on exceptions
├── Data/
│   ├── Config.xlsx                  # Configuration file (Settings, Constants, Assets)
│   ├── AllReport.xlsx               # Consolidated processing report
│   ├── Input/                       # Downloaded input Excel files
│   ├── Output/                      # Output files
│   └── Temp/                        # Temporary working files
├── Documentation/                   # Project documentation
├── Exceptions_Screenshots/          # Exception screenshots storage
├── Tests/                           # Unit & integration test cases
├── project.json                     # UiPath project manifest
└── LICENSE                          # MIT License
```

---

## 🔧 Prerequisites

- **UiPath Studio** `2026.0` or later (Windows target framework)
- **UiPath Orchestrator** with Queue configured
- **Microsoft Edge** browser
- **Email Account** with IMAP/SMTP access (e.g., Gmail with App Password)
- **Orchestrator Assets**:
  - Credential asset for email authentication
  - Queue for transaction processing

---

## 🛠️ Configuration

All settings are managed via `Data/Config.xlsx`:

### Settings Sheet

| Setting | Description |
|---|---|
| `OrchestratorQueueName` | Name of the Orchestrator queue |
| `OrchestratorQueueFolder` | Folder path in Orchestrator |
| `BrowserUrl` | URL of the password reset web portal |
| `InputExcelFolderName` | Path for input Excel files |
| `InputExcelFileName` | Name of the input Excel file |
| `SheetName` | Sheet name in the input Excel |
| `Gmail_IMAP_Server` | IMAP server address |
| `Gmail_IMAP_Port` | IMAP port number |
| `Gmail_SMTP_Server` | SMTP server address |
| `Gmail_SMTP_Port` | SMTP port number |
| `Gmail_Folder_Read` | Email folder to read from |
| `Gmail_IMAP_Limit` | Max number of emails to read |
| `Reciver_Subject` | Subject line filter for incoming emails |
| `Reciver_Gmail` | Recipient email for output delivery |
| `Subject` | Subject prefix for outgoing emails |
| `OrchstratorCredentialFileName` | Orchestrator credential asset name |
| `SortedDataFile` | Path for the AllReport output file |

---

## 🚀 Setup & Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/<your-username>/REF_PasswordReset.git
   ```

2. **Open in UiPath Studio**
   - Open `project.json` in UiPath Studio 2026.0+
   - Restore NuGet packages

3. **Configure Orchestrator**
   - Create an Orchestrator Queue with the name specified in `Config.xlsx`
   - Create a Credential asset with email credentials

4. **Update `Data/Config.xlsx`**
   - Fill in email server details (IMAP/SMTP)
   - Set the browser URL for the password reset portal
   - Configure email filtering settings

5. **Prepare Input**
   - Send an email with an Excel attachment containing columns: `UName`, `EmpCode`, `Email`
   - The email subject must match the configured `Reciver_Subject`

6. **Run the process**
   - Execute `Main.xaml` from UiPath Studio or publish to Orchestrator

---

## 🧪 Test Cases

The project includes the following test cases in the `Tests/` directory:

| Test Case | Description |
|---|---|
| `MainTestCase.xaml` | End-to-end test of the full process |
| `InitAllSettingsTestCase.xaml` | Tests configuration loading |
| `InitAllApplicationsTestCase.xaml` | Tests application initialization |
| `GetTransactionDataTestCase.xaml` | Tests queue item retrieval |
| `ProcessTestCase.xaml` | Tests core business logic |
| `WorkflowTestCaseTemplate.xaml` | Template for creating new test cases |

---

## 📦 Dependencies

| Package | Version |
|---|---|
| `UiPath.System.Activities` | 26.2.0 |
| `UiPath.UIAutomation.Activities` | 26.1.0-preview |
| `UiPath.Excel.Activities` | 3.4.1 |
| `UiPath.Mail.Activities` | 2.7.10 |
| `UiPath.Azure.Activities` | 1.6.3 |
| `UiPath.Testing.Activities` | 25.10.1 |

---

## 📝 Input Excel Format

The input Excel file should contain the following columns:

| Column | Description | Required |
|---|---|---|
| `UName` | Employee username | ✅ Yes |
| `EmpCode` | Employee code | ✅ Yes |
| `Email` | Employee email address | ✅ Yes |

> **Note:** Records with missing required fields are filtered out by the Dispatcher and flagged in the report.

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'Add your feature'`)
4. Push to the branch (`git push origin feature/your-feature`)
5. Open a Pull Request

---

<p align="center">
  Built with ❤️ using <strong>UiPath Robotic Enterprise Framework</strong>
</p>
