# Email Notification Agent

An AI-powered expense claim agent built with **Azure AI Agent Service** and the **Microsoft Agent Framework SDK**. The agent reads expense data, processes a user instruction, and automatically composes and submits an expense claim email.

---

## Demo

```
Here is the expenses data in your file:

date,description,amount
07-Mar-2025,taxi,24.00
07-Mar-2025,dinner,65.50
07-Mar-2025,hotel,125.90

What would you like me to do with it?
> Submit an expense claim

To: expenses@contoso.com
Subject: Expense Claim

Dear Expenses Team,

Please find my expense claim for reimbursement:

- 07-Mar-2025  Taxi      $24.00
- 07-Mar-2025  Dinner    $65.50
- 07-Mar-2025  Hotel    $125.90

Total: $215.40

# Agent:
I've submitted your expense claim via email to expenses@contoso.com.
```

---

## Tech Stack

| Component | Technology |
|---|---|
| AI Agent | Microsoft Agent Framework SDK (`agent-framework`) |
| Model | GPT-4.1 via Azure AI Foundry |
| Authentication | Azure CLI Credential (async) |
| Language | Python 3.13+ |
| Cloud | Azure AI Agent Service |

---

## Project Structure

```
Labfiles/07-agent-framework/python/
├── agent-framework.py   # Main agent application
├── data.txt             # Sample expense data (CSV)
├── requirements.txt     # Python dependencies
├── .env                 # Project config (gitignored)
└── README.md            # Lab-specific setup guide
```

---

## Quick Start

### Prerequisites

- Python 3.13+
- Azure CLI (`brew install azure-cli` on macOS)
- Azure subscription with AI Foundry project deployed
- VS Code + Microsoft Foundry extension

### 1. Clone the repo

```bash
git clone https://github.com/shashankkallahallisuresh/Email-Notification-agent.git
cd Email-Notification-agent/Labfiles/07-agent-framework/python
```

### 2. Set up virtual environment

```bash
python -m venv labenv
source labenv/bin/activate        # macOS/Linux
# .\labenv\Scripts\Activate.ps1  # Windows
pip install -r requirements.txt
```

### 3. Configure environment variables

Create a `.env` file in the `python/` folder:

```env
PROJECT_ENDPOINT="https://<your-azure-foundry-project-endpoint>"
MODEL_DEPLOYMENT_NAME="gpt-4.1"
```

Get the `PROJECT_ENDPOINT` by right-clicking your project in the Microsoft Foundry VS Code extension → **Copy Project Endpoint**.

### 4. Log in to Azure

```bash
az login
```

### 5. Run the app

```bash
python agent-framework.py
```

When prompted, type:
```
Submit an expense claim
```

---

## Azure Setup Steps

### Step 1 — Install Microsoft Foundry VS Code Extension

- VS Code → Extensions → Search **Microsoft Foundry** → Install
- Sign in to Azure from the extension sidebar

### Step 2 — Create a Foundry Project

- Click **+** next to Resources in the Foundry extension
- Select subscription → create/select resource group → enter project name
- Wait for "Project deployed successfully"

### Step 3 — Register Container Registry Provider

If you see `Microsoft.ContainerRegistry` not registered error:

```bash
az provider register --namespace Microsoft.ContainerRegistry --wait
az provider show --namespace Microsoft.ContainerRegistry --query "registrationState"
# → "Registered"
```

### Step 4 — Deploy GPT-4.1 Model

- Click **+** next to Models in the Foundry extension
- Select **gpt-4.1** → Deployment type: **Global Standard** → Deploy

### Step 5 — Copy Project Endpoint

- Right-click your project deployment → **Copy Project Endpoint**
- Paste into `.env` as `PROJECT_ENDPOINT`

---

## Key Bug Fixes Applied

| Issue | Root Cause | Fix |
|---|---|---|
| `NameError: Agent` | `Agent` not imported | `from agent_framework import tool, Agent` |
| `.env` values not loading | `load_dotenv()` never called | Added `from dotenv import load_dotenv` + `load_dotenv()` |
| `try` block out of scope | Incorrect indentation outside `async with` | Indented `try` block inside `async with` body |
| `TypeError: async context manager` | `azure.identity.AzureCliCredential` is sync-only | Switched to `from azure.identity.aio import AzureCliCredential` |
| Azure CLI not found | CLI not installed | `brew install azure-cli` + `az login` |
| Container registry error | Resource provider not registered | `az provider register --namespace Microsoft.ContainerRegistry` |

---

## How It Works

```
agent-framework.py
│
├── load_dotenv()                    # Load PROJECT_ENDPOINT and MODEL_DEPLOYMENT_NAME
├── main()
│   ├── Read data.txt                # Load expense CSV
│   └── await process_expenses_data()
│
├── process_expenses_data()
│   └── async with Agent(
│         client=AzureOpenAIResponsesClient   # Connects to Azure AI Foundry
│         instructions="..."                  # System prompt for expense claims
│         tools=[submit_claim]                # Register custom tool
│       ):
│         agent.run(prompt + expenses_data)   # Invoke agent
│
└── @tool submit_claim()             # Custom tool: prints email to console
```

---

## Cleanup

After completing the exercise, delete Azure resources to avoid charges:

1. **Delete model** — Foundry extension → Models → right-click → Delete
2. **Delete resource group** — Azure Portal → Resource Groups → Delete

---

## References

- [Microsoft Agent Framework SDK](https://aka.ms/agent-framework)
- [Azure AI Agent Service Docs](https://aka.ms/azure-ai-agent-service)
- [Azure AI Foundry](https://ai.azure.com)
- [Original MS Learn Lab](https://microsoftlearning.github.io/mslearn-ai-agents)

