# Connect Claude Code to Anthropic Models via AWS Bedrock

## Step 1 — Sign in and pick a region that supports Claude

1. Go to **https://console.aws.amazon.com** and sign in.
2. Bedrock and the Claude models are not available in every AWS region. Easy choices:
   - **`us-east-1`** (N. Virginia) — widest model availability
   - **`us-west-2`** (Oregon) — also good
3. Look at the top-right of the AWS Console — there's a region dropdown. Click it and choose **US East (N. Virginia) us-east-1**.

Write down the region code (`us-east-1`). You'll need it later.

---

## Step 2 — Request access to the Claude models in Bedrock

AWS doesn't enable model access automatically — you have to opt in to each model.

1. In the AWS Console search bar at the top, type **"Bedrock"** and click the result.
2. In the left sidebar, click **"Model access"** (near the bottom).
3. Click the orange **"Modify model access"** (or "Manage model access") button.
4. Find **Anthropic** in the provider list. Check the boxes for the Claude models you want, for example:
   - **Claude Opus 4.7**
   - **Claude Sonnet 4.6**
   - **Claude Haiku 4.5**
5. Click **Next**, fill in the short use-case form (one sentence is fine, e.g. *"Internal developer tooling for our team"*), and submit.
6. Access for Anthropic models is usually granted within a few minutes. Refresh the page until the status shows **"Access granted"** (green).

> If a model says **"Available to request"** and you don't see a checkbox, your account may need higher permissions — ask your AWS admin.

---

## Step 3 — Create an access key (so your terminal can talk to AWS)

If you already have AWS credentials configured on your machine (you can check by running `aws sts get-caller-identity` in a terminal — if it returns your account info, **skip to Step 5**), you can reuse those as long as the user has Bedrock access. Otherwise:

1. In the AWS Console search bar, type **"IAM"** and click the result.
2. In the left sidebar, click **"Users"**, then **"Create user"**.
3. Name the user something memorable like `claude-code-user`. Click **Next**.
4. On the permissions page, choose **"Attach policies directly"**.
5. In the search box, type `Bedrock`. Check the box next to **`AmazonBedrockFullAccess`**.
6. Click **Next**, then **Create user**.
7. Click into the user you just made. Go to the **"Security credentials"** tab.
8. Scroll to **"Access keys"** → click **"Create access key"**.
9. Pick **"Command Line Interface (CLI)"**, acknowledge the warning, click **Next**, then **Create access key**.
10. You'll see two values:
    - **Access key ID** (starts with `AKIA…`)
    - **Secret access key** (a long random string)

**Copy both somewhere safe immediately.** AWS only shows the secret key once. A password manager (1Password, Bitwarden) is ideal.

---

## Step 4 — Install the AWS CLI on your computer

Skip this step if `aws --version` already prints a version number in your terminal.

### Mac
1. Open the **Terminal** app (press `Cmd + Space`, type "Terminal", hit Enter).
2. Paste this and press Enter:
   ```bash
   curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg" && sudo installer -pkg AWSCLIV2.pkg -target /
   ```
3. Enter your Mac password when prompted.

### Windows
1. Download the installer from **https://awscli.amazonaws.com/AWSCLIV2.msi**.
2. Double-click it and follow the wizard.
3. Open **PowerShell** (search for it in the Start menu) for the next step.

### Linux
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

### Verify it installed
In your terminal, run:
```bash
aws --version
```
You should see something like `aws-cli/2.x.x`.

---

## Step 5 — Save your AWS credentials on your computer

In a terminal, run:

```bash
aws configure
```

It'll ask you four things:

| Prompt | What to enter |
|---|---|
| AWS Access Key ID | Paste the **Access key ID** from Step 3 |
| AWS Secret Access Key | Paste the **Secret access key** from Step 3 |
| Default region name | `us-east-1` (or whatever you chose in Step 1) |
| Default output format | `json` |

Test that it worked:
```bash
aws sts get-caller-identity
```
You should see your AWS account number and the user name `claude-code-user`. If you do, credentials are wired up correctly.

> Already have other AWS profiles on this machine? You can keep them separate by running `aws configure --profile claude-bedrock` instead, then later setting `export AWS_PROFILE=claude-bedrock` before launching Claude.

---

## Step 6 — Install Claude Code

Claude Code needs **Node.js** (a free runtime) installed first.

### Install Node.js
Skip if `node --version` already prints `v20.x.x` or higher.

- **Mac:** Go to **https://nodejs.org**, click the green LTS button, run the installer.
- **Windows:** Same — **https://nodejs.org**, LTS, run the installer.
- **Linux:** Use your package manager, e.g. `sudo apt install nodejs npm`.

### Install Claude Code itself
In your terminal:
```bash
npm install -g @anthropic-ai/claude-code
```

Verify:
```bash
claude --version
```

---

## Step 7 — Tell Claude Code to use Bedrock

Claude Code talks to Anthropic's API by default. Two environment variables flip it over to Bedrock:

### Mac / Linux
Open your shell config file. For most Macs that's `~/.zshrc`:
```bash
open -e ~/.zshrc
```
(On Linux, use `nano ~/.bashrc` instead.)

Add these two lines to the bottom of the file:
```bash
export CLAUDE_CODE_USE_BEDROCK=1
export AWS_REGION=us-east-1
```

Save and close, then reload:
```bash
source ~/.zshrc
```

### Windows (PowerShell)
Run:
```powershell
[Environment]::SetEnvironmentVariable("CLAUDE_CODE_USE_BEDROCK", "1", "User")
[Environment]::SetEnvironmentVariable("AWS_REGION", "us-east-1", "User")
```
Then **close and reopen PowerShell** for the changes to take effect.

---

## Step 8 — Run Claude Code

In your terminal, navigate to any folder you want to work in:
```bash
cd ~/Documents
```

Then start Claude Code:
```bash
claude
```

If everything is wired up correctly, Claude will start up and respond to your prompts — just like the regular version, but every request is going through your AWS Bedrock account.

Try asking it something simple like:
> "What model are you running on?"

If you see Claude respond, **you're done.** 🎉

---

## Troubleshooting

### "Could not load credentials from any providers"
Your AWS CLI isn't configured. Re-run `aws configure` and double-check Step 5.

### "AccessDeniedException" or "You don't have access to the model"
You haven't enabled the specific Claude model in Bedrock yet. Go back to **Step 2** and make sure the model status is **"Access granted"**. If you're using a different AWS profile, also confirm that profile's user has the `AmazonBedrockFullAccess` policy.

### "Region not supported"
Bedrock isn't enabled in the region you set. Change `AWS_REGION` to `us-east-1` or `us-west-2`.

### "Command not found: claude"
Node.js didn't install Claude Code to a folder your terminal can see. Try closing and reopening your terminal first. If it still doesn't work, run `npm install -g @anthropic-ai/claude-code` again and watch for errors.

### Claude is using the wrong model
You can pin a specific model by adding one more environment variable. For example, to force Sonnet 4.6:
```bash
export ANTHROPIC_MODEL=anthropic.claude-sonnet-4-6-20251022-v1:0
```
(Exact model IDs are listed in the Bedrock console under **Model access**.)

---

## A note on cost

Bedrock charges per million input/output tokens, billed to your AWS account. Pricing is published at **https://aws.amazon.com/bedrock/pricing/**. For light personal use you'll typically spend a few dollars per month; heavy use can run higher. Set up an **AWS Budget alert** if you want a safety net:

1. AWS Console → search **"Budgets"** → **Create budget**.
2. Pick **"Monthly cost budget"**, set a dollar amount, add your email.

---

## What you've accomplished

- ✅ Claude models enabled in Bedrock under your AWS account
- ✅ AWS credentials saved on your computer
- ✅ Claude Code installed
- ✅ Claude Code talking to Anthropic models via your own AWS account

All future sessions are just: open terminal → `claude`.
