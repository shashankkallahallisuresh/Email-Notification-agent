# Email Notification Agent — Azure AI Agent Service + Microsoft Agent Framework

An AI agent that processes expense data and automatically submits an expense claim by composing and sending an email. Built using **Azure AI Agent Service** and the **Microsoft Agent Framework SDK**.

---

## What It Does

1. Reads expense data from a local `data.txt` CSV file
2. Asks the user what to do with the data
3. The AI agent processes the instruction, itemizes the expenses, and calls a custom `submit_claim` tool
4. The tool simulates sending an email to `expenses@contoso.com` with an itemized expense report

---

## Architecture

```
agent-framework.py
│
├── main()                          # Loads data.txt, prompts user
│
├── process_expenses_data()         # Creates Agent with:
│   ├── AzureOpenAIResponsesClient  # Connects to Azure AI Foundry project
│   ├── AzureCliCredential (async)  # Authenticates via Azure CLI
│   └── submit_claim tool           # Custom tool for email simulation
│
└── submit_claim()                  # @tool — prints email to console
```

---

## Prerequisites

- Python 3.13+
- Azure CLI installed and logged in (`az login`)
- An active Azure subscription
- Visual Studio Code with the **Microsoft Foundry** extension
- Git

---

## Setup Steps

### 1. Install the Microsoft Foundry VS Code Extension

- Open VS Code → Extensions (`Ctrl+Shift+X`)
- Search for **Microsoft Foundry** and install it
- Sign in to Azure using the extension sidebar

### 2. Create a Foundry Project

- In the Foundry extension sidebar, click **+** next to Resources
- Select your Azure subscription
- Create a new resource group (e.g., `rg-ai-agents-lab`) or use an existing one
- Enter a project name (e.g., `ai-agents-project`) and wait for deployment

### 3. Register the Container Registry Provider

If you see `The subscription is not registered to use namespace 'Microsoft.ContainerRegistry'`, run:

```bash
az provider register --namespace Microsoft.ContainerRegistry --wait
```

Then verify:

```bash
az provider show --namespace Microsoft.ContainerRegistry --query "registrationState"
# Should output: "Registered"
```

### 4. Deploy a Model

- In the Foundry extension, click **+** next to Models
- Select **gpt-4.1** from the Model Catalog
- Set deployment name to `gpt-4.1`, type to **Global Standard**
- Click Deploy

### 5. Get the Project Endpoint

- Right-click your project deployment in the Foundry extension sidebar
- Select **Copy Project Endpoint**

### 6. Clone the Repository

```bash
git clone https://github.com/MicrosoftLearning/mslearn-ai-agents.git
cd mslearn-ai-agents/Labfiles/07-agent-framework/python
```

### 7. Set Up Python Virtual Environment

```bash
python -m venv labenv
source labenv/bin/activate        # macOS/Linux
# .\labenv\Scripts\Activate.ps1  # Windows PowerShell

pip install -r requirements.txt
```

### 8. Configure Environment Variables

Create/edit the `.env` file:

```env
PROJECT_ENDPOINT="https://<your-project-endpoint>"
MODEL_DEPLOYMENT_NAME="gpt-4.1"
```

Replace `<your-project-endpoint>` with the URL copied in Step 5.

### 9. Install Azure CLI (if not already installed)

```bash
brew install azure-cli    # macOS
```

Then log in:

```bash
az login
```

---

## Running the App

```bash
cd Labfiles/07-agent-framework/python
source labenv/bin/activate
python agent-framework.py
```

When prompted:
```
What would you like me to do with it?
> Submit an expense claim
```

### Expected Output

```
To: expenses@contoso.com
Subject: Expense Claim
Dear Expenses Team,

Please find below my expense claim for reimbursement:

- 07-Mar-2025  Taxi     $24.00
- 07-Mar-2025  Dinner   $65.50
- 07-Mar-2025  Hotel   $125.90

Total: $215.40

# Agent:
I've submitted your expense claim via email to expenses@contoso.com ...
```

---

## Project Structure

```
Labfiles/07-agent-framework/python/
├── agent-framework.py   # Main agent application
├── data.txt             # Sample expense data (CSV)
├── requirements.txt     # Python dependencies
├── .env                 # Project endpoint config (gitignored)
└── README.md            # This file
```

### Key Dependencies (`requirements.txt`)

| Package | Purpose |
|---|---|
| `agent-framework` | Microsoft Agent Framework SDK |
| `azure-identity` | Azure authentication (AzureCliCredential) |
| `python-dotenv` | Load `.env` configuration |
| `pydantic` | Field annotations for tool parameters |
| `opentelemetry-semantic-conventions-ai` | AI observability |

---

## Key Code Decisions & Bug Fixes

During setup, the following issues were identified and resolved:

| Issue | Fix |
|---|---|
| `Agent` not imported | Added `Agent` to `from agent_framework import tool, Agent` |
| `.env` not loaded | Added `from dotenv import load_dotenv` and `load_dotenv()` |
| `try` block outside `async with` | Indented `try` block inside the `async with` context |
| `AzureCliCredential` doesn't support `async with` | Switched to `from azure.identity.aio import AzureCliCredential` |
| Azure CLI not found | Installed via `brew install azure-cli` |
| `Microsoft.ContainerRegistry` not registered | Registered via `az provider register` |

---

## Cleanup

To avoid ongoing Azure charges after completing the exercise:

1. **Delete the model** — Right-click the deployed model in the Foundry extension → Delete
2. **Delete the resource group** — Azure Portal → Resource Groups → select group → Delete resource group

---

## References

- [Microsoft Agent Framework SDK](https://aka.ms/agent-framework)
- [Azure AI Agent Service](https://aka.ms/azure-ai-agent-service)
- [Azure AI Foundry](https://ai.azure.com)
- [Original Lab Exercise](https://microsoftlearning.github.io/mslearn-ai-agents)
