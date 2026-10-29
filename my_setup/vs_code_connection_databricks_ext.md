# Tip : Rule of thumb: never use spaces in Databricks profile names.

I have used workspace name as "sulara"
see under "CURRENT WORKSPACE" on top right corner of your databricks UI

you can see all the connection data in .databricks config file shown as .databrickscfg
I think this file is saved on C:users/..bla bla bla

here is how data inside the file

Exampleaa:

; The profile defined in the DEFAULT section is to be used as a fallback when no profile is explicitly specified.
[DEFAULT]

[sulara]
host         = https://dbc-4ce3f3ad-bb59.cloud.databricks.com
account_id   = e8b971bc-e1d8-4f26-879a-1c60d40f3562
workspace_id = 2649178100489795
auth_type    = databricks-cli

[__settings__]
default_profile = sulara
auth_storage    = secure





# Databricks Official VS Code Extension - this has all the tools + CLI to connect (EASY)
The standard way is the official Databricks extension for VS Code with OAuth login. Free Edition only has **serverless** compute, so you point everything at serverless instead of a cluster.


## Minimal setup

**1. Install the extension**

In VS Code Extensions, search **"Databricks"** (publisher: Databricks) and install it. It includes the Databricks CLI, so you don't need a separate install.

**2. Connect to your workspace**

- Open your project folder in VS Code.
- Click the Databricks icon in the sidebar, then **SSH Tunnel** --> **Start SSH Tunnel**.
- Paste your workspace URL. It looks like `https://dbc-xxxxxxxx-xxxx.cloud.databricks.com`, and you can copy it from your browser when you're logged into Free Edition.
or Paste this ULR if you want to skip the profile asking: 
https://dbc-xxxxxxxx-xxxx.cloud.databricks.com --profile free
- Choose **OAuth (user to machine)**. A browser tab opens. Log in, and you're done.



This saves a profile to `~/.databrickscfg`, which the CLI, SDK and Databricks Connect all reuse.

**3. Pick compute**

Under Compute in the extension, select **Serverless**. Free Edition doesn't allow classic clusters, so that's the only choice.

# Option 2

## Auth using "Access tokens" in databricks
Go to setting in databricks --> Settings --> Developer --> Access tokens --> Manage --> Generate new token --> Copy the token
Then select "Personal Access Token" from the vs code top panel
type "sulara" as the name for the new profile
paste the token --> Enter


## Common gotchas

- **Version mismatch.** If you get `databricks-connect` errors, the usual cause is a Python or package version that doesn't match the serverless runtime. Pin `databricks-connect` to the version the extension suggests.
- **Unity Catalog only.** Free Edition only has Unity Catalog, so use three-level names (`workspace.default.my_table`) and volumes, not DBFS paths.
- **Do your syncing through Git.** For moving notebooks and code, Git folders in the workspace (or Asset Bundles later) are cleaner than the extension's file sync.

That's all you need for your retail lakehouse project. If you'd like to deploy jobs from VS Code later, Databricks Asset Bundles (`databricks bundle init`) is the next step.


also need to ask, why vscode opens a whole separate workspace for databricks. then I loose all my default installed extension on the newly opened vscode

That’s how VS Code’s Remote - SSH works in general, not something specific to Databricks.

Why a separate window opens

A remote window is split in two:

The UI (themes, keybindings, the editor itself) stays on your PC.
Everything else (file access, terminals, debuggers, language servers, and most extensions) runs on the remote machine, here the Databricks compute.

So VS Code opens a new window tied to that remote. Your extensions aren’t gone. They’re still installed on your PC, but most of them need to run where the code lives, so they have to be installed again on the remote side. That’s why the Extensions panel shows them under “Local – Installed” with an Install in SSH: … button.

If you do use remote sessions, you can bring your extensions along

One-off: in the Extensions view of the remote window, click the cloud-download icon labelled Install Local Extensions in ‘SSH: …’ and select them all.
Automatic: add the extensions you always want to your settings.json:
json
"remote.SSH.defaultExtensions": [
  "ms-python.python",
  "ms-toolsai.jupyter",
  "github.copilot"
]

Bear in mind each remote install has to download over the same slow connection you saw earlier, and the Databricks compute may get recycled, so you could end up reinstalling often.

For your workflow, this is another reason to skip SSH

With the GitHub-centric setup, you open the cloned repo in a normal local VS Code window. All your extensions, settings and terminal stay as they are, and nothing has to download. Run and test on the Databricks side through the Git folder. Later, the CLI and Databricks Connect also run locally, in the same window.




# --------------------------------------------------------------------------------------



# CLI

Install Databricks CLI using winget for Windows

# Steps
- open powershell or cmd
- check databricks is already installed
- >databricks --version
- if not, then 
- >winget install Databricks.DatabricksCLI
- then check
- >databricks --version
- it will show the version like "Databricks CLI v1.21.0"

# Access databricks workspace from CMD
- >databricks auth login --host https://dbc-4ce3f3ad-bb59.cloud.databricks.com # this is your url of databricks free edition when you open dtabricks in web browser


C:\Users\sular>databricks auth login --host https://dbc-4ce3f3ad-bb59.cloud.databricks.com
Databricks profile name [dbc-4ce3f3ad-bb59]: Sulara Perera
Profile Sulara Perera was successfully saved

# To see the workspace files
- >databricks workspace list /Users/sulara.perera@outlook.com -p "Sulara Perera"


'''
ID                Type       Language  Path
2804603448025749  DIRECTORY            /Users/sulara.perera@outlook.com/.db_internal
2465236612293661  DIRECTORY            /Users/sulara.perera@outlook.com/1.setup
1181924493283571  DIRECTORY            /Users/sulara.perera@outlook.com/projects
796797261560992   DIRECTORY            /Users/sulara.perera@outlook.com/Drafts
2804603448025750  DIRECTORY            /Users/sulara.perera@outlook.com/practice
2336653834907885  DIRECTORY            /Users/sulara.perera@outlook.com/.assistant
737346632746696   DIRECTORY            /Users/sulara.perera@outlook.com/.databricks

'''

# To see catalogs
>databricks catalogs list -p "Sulara Perera"


# Use case of using CLI

The browser is for clicking around. The CLI is for **automation and repeatability**, which real data engineering depends on. Here's what it lets you do that the UI can't do well:

- **Deploy code like software (Asset Bundles):** you keep notebooks, jobs and pipelines as files in Git on your laptop, then run `databricks bundle deploy` to push everything to dev, test or prod in one go. This is the modern standard way to do CI/CD on Databricks.
- **CI/CD pipelines:** GitHub Actions or Azure DevOps use the same CLI to deploy automatically when you merge code.
- **Scripting and bulk work:** upload 100 files to a Volume, start or stop clusters, or trigger and monitor jobs with one command instead of many clicks.
- **Local development:** you write code in VS Code or another editor and sync or run it against your workspace.
- **Admin tasks:** manage users, permissions, secrets and tokens from scripts. Some features, like secrets, are easiest through the CLI.
- **AI tooling:** tools like Claude Code can drive your workspace through the CLI on your behalf.

In short, logging in today set up the connection between your machine and the workspace. Your next step on that path would be to try `databricks bundle init` and build your first Asset Bundle. Want me to walk you through it?
