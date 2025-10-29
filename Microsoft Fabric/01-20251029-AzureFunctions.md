# 🚀 Run Azure Functions from Microsoft Fabric Securely using Managed Private Endpoints

> This approach allows you to run **Azure Functions - HTTP Trigger** from **Microsoft Fabric** securely **without requiring any inbound public network access or a VNET gateway**.  
> Communication happens privately through **Managed Private Endpoints** — keeping your data secure and isolated.

## 🧠 Overview

This solution integrates:
- **Microsoft Fabric Notebooks**
- **Azure Functions**
- **Azure Key Vault**

All within a **private network**, ensuring no public exposure and seamless secret management.

## 🧩 Why Use This Setup?

✅ No inbound public network access rules or VNET gateway  
✅ Fully private communication using Managed Private Endpoints  
✅ Secrets securely stored and retrieved from Azure Key Vault  
✅ Simple automation via Fabric Pipelines  

## ⚙️ Prerequisites

Before you begin, make sure the following are ready:

1. Create **Azure Function App**
   - An **HTTP-triggered** Azure Function App is created.
   - Configure Azure Functions app to use **[Microsoft Entra sign-in](https://learn.microsoft.com/en-gb/azure/app-service/configure-authentication-provider-aad?tabs=workforce-configuration)**.
   - The Function App uses a **function key** for authentication.
2. Create **Azure Key Vault**
3. Create **Managed Private Endpoints** from your Fabric Workspace to:
   - Azure Function App  
   - Azure Key Vault  
4. Create **Fabric Workspace Managed Identity**  
   (Used to authenticate and retrieve secrets securely.)


## 🔐 Azure Functions and Azure Key Vault Configuration

**Collect the following details from your Azure Function App - HTTP Trigger:**
- Function URL
- Token Audience (e.g., `api://<client-id>`)
- Tenant ID
- Client ID
- Client Secret
- Function Key

**Collect the following details from your Azure Key Vault**:
1. Create secrets to store:
   - `Client Secret`
   - `Function Key`
2. Copy the **secret names**.
3. Copy your **Key Vault URL**.
4. Grant your **Fabric Workspace Identity** the role:  
   `Key Vault Secrets User`

## 🧮 Fabric Notebook Setup

1. In your **Fabric Workspace**, Create a new notebook named:  
   `NB_FUNCTION_APP_HTTP_TRIGGER`

2. Paste the following Python code in a notebook cell:

```python
# ─── REQUIRED INPUTS ────────────────────────────────────────────────────────────
FUNCTION_URL        = ""  # full function URL (incl. route/query if any)
TOKEN_AUDIENCE      = ""  # the 'audience' / resource
TENANT_ID           = ""
CLIENT_ID           = ""
KEYVAULT_URL        = ""
SECRETNAME_CLIENT_SECRET = ""
SECRETNAME_FUNCTION_KEY   = ""
```

When orchestrating the notebook using a Fabric pipeline, ensure that the parameters listed above are set to **"Toggle parameter"** in the pipeline configuration. This allows the pipeline to pass values dynamically to the notebook.

3. Paste the remaining Python code in another notebook cell:

```python
# ─── RETRIEVE SECRETS ───────────────────────────────────────────────────────────
CLIENT_SECRET = notebookutils.credentials.getSecret(
    f"{KEYVAULT_URL}",
    f"{SECRETNAME_CLIENT_SECRET}"
)

FUNCTION_KEY = notebookutils.credentials.getSecret(
    f"{KEYVAULT_URL}",
    f"{SECRETNAME_FUNCTION_KEY}"
)

# ─── INSTALL REQUIRED LIBRARIES ─────────────────────────────────────────────────
#%pip install --quiet msal requests

import msal
import requests

# --- GET TOKEN ---
def get_token(tenant_id, client_id, client_secret, audience):
    app = msal.ConfidentialClientApplication(
        client_id,
        authority=f"https://login.microsoftonline.com/{tenant_id}",
        client_credential=client_secret,
    )
    scope = [audience.rstrip('/') + "/.default"]
    token = app.acquire_token_for_client(scopes=scope)
    if "access_token" not in token:
        raise Exception(f"Token error: {token}")
    return token["access_token"]

# --- CALL FUNCTION ---
def call_function(url, function_key, token, method="GET", data=None):
    headers = {
        "Authorization": f"Bearer {token}",
        "x-functions-key": f"{function_key}",
        "Content-Type": "application/json"
    }
    if method == "POST":
        resp = requests.post(url, headers=headers, json=data)
    else:
        resp = requests.get(url, headers=headers)
    return resp

# --- RUN ---
token = get_token(TENANT_ID, CLIENT_ID, CLIENT_SECRET, TOKEN_AUDIENCE)
response = call_function(FUNCTION_URL, FUNCTION_KEY, token)

print("Status code:", response.status_code)
print("Response:", response.text)
````

At this point, you can **run the notebook** by entering the parameter values to test it, or proceed to create a Fabric pipeline to orchestrate the notebook.


## 🔄 Create a Fabric Pipeline

1. Create a new **Pipeline** in Fabric.
2. Add a **Notebook Activity**.
3. Select the notebook: `NB_FUNCTION_APP_HTTP_TRIGGER`.
4. Add the following **base parameters** as strings:

| Parameter                  | Description                                       |
| -------------------------- | ------------------------------------------------- |
| `FUNCTION_URL`             | Full Azure Function URL                           |
| `TOKEN_AUDIENCE`           | Token resource audience (e.g., api://<client-id>) |
| `TENANT_ID`                | Tenant ID                                |
| `CLIENT_ID`                | Client ID                        |
| `KEYVAULT_URL`             | Azure Key Vault URL                               |
| `SECRETNAME_CLIENT_SECRET` | Secret name for Client Secret                     |
| `SECRETNAME_FUNCTION_KEY`  | Secret name for Function Key                      |

5. **Save** the pipeline.
6. Perform a **test run** — you should see your Azure Function executed successfully through private endpoints.


## 📚 References

* [Microsoft Fabric Workspace Identity](https://learn.microsoft.com/en-gb/fabric/security/workspace-identity)
* [Microsoft Fabric Create Managed Private Endpoints](https://learn.microsoft.com/en-us/fabric/security/security-managed-private-endpoints-create)


## ☕ Support the Project

If you find these guides helpful, consider supporting my work so I can keep brewing more content!

[![Buy Me a Tea](https://img.buymeacoffee.com/button-api/?text=Buy%20me%20a%20tea&emoji=🍵&slug=teawithdata&button_colour=FFDD00&font_colour=000000&font_family=Poppins&outline_colour=000000&coffee_colour=ffffff)](https://www.buymeacoffee.com/teawithdata)

### 🪶 Author
**Tea With Data**  
Sharing the art of data with a cup of tea.

[![carolinetwd profile views](https://u8views.com/api/v1/github/profiles/229321296/views/day-week-month-total-count.svg)](https://u8views.com/github/carolinetwd)