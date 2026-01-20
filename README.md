Guide: Setting up Claude Code with Vertex AI (Service Account Method)
This guide walks through setting up a persistent authentication method for Claude Code on Vertex AI using a Google Cloud Service Account. This avoids the need for daily re-authentication.

1. Install and Initialize Google Cloud CLI
Installation
On macOS, use Homebrew:

Bash
brew install --cask gcloud-cli
Python Troubleshooting
If you encounter Python version issues, ensure you are not in a Conda environment and try updating Python:

Bash
brew install python@3.13
If gcloud cannot find Python, set the environment variable (ensure which python3 returns a valid path first):

Bash
export CLOUDSDK_PYTHON=$(which python3)
Initial Authentication
Login to your primary Google account and set your project:

Bash
# Authenticate with your Google Cloud account
gcloud auth login

# Configure your project
gcloud config set project bryan-usage-0

# Authenticate for local applications (one-time setup)
gcloud auth application-default login
2. Create a Persistent Service Account
A service account acts as a dedicated "bot" identity for Claude Code that does not expire like a standard user session.

Create the Identity
Replace INSERT_NAME and INSERT_DISPLAY_NAME with your preferred identifiers (e.g., claude-bot).

Bash
gcloud iam service-accounts create INSERT_NAME \
    --description="Service account for Vertex AI" \
    --display-name="INSERT_DISPLAY_NAME"
Grant Permissions
Assign the Vertex AI User role to the service account so it can call the Claude models:

Bash
gcloud projects add-iam-policy-binding bryan-usage-0 \
    --member="serviceAccount:INSERT_NAME@bryan-usage-0.iam.gserviceaccount.com" \
    --role="roles/aiplatform.user"
Generate the JSON Key File
This command creates a local file that contains the credentials.

Bash
gcloud iam service-accounts keys create ~/claude-sa-key.json \
    --iam-account=INSERT_NAME@bryan-usage-0.iam.gserviceaccount.com
Security Note: Keep ~/claude-sa-key.json private. Add it to your .gitignore if you are working within a git repository.

3. Configure Your Shell (.zshrc or .bashrc)
Add the following block to the bottom of your ~/.zshrc (macOS/Zsh) or ~/.bashrc (Linux/Bash) file to automate the authentication every time you run Claude.

Bash
##### START Google Cloud VertexAI API for Claude Code Setup #####
# Path to the service account key created in the previous step
export CLAUDE_SA_KEY="$HOME/claude-sa-key.json"

# Alias to intercept 'claude' and inject the service account credentials
alias claude='GOOGLE_APPLICATION_CREDENTIALS="$CLAUDE_SA_KEY" claude'

# Enable Vertex AI integration settings
export CLAUDE_CODE_USE_VERTEX=1
export ANTHROPIC_VERTEX_PROJECT_ID=bryan-usage-0
export ANTHROPIC_VERTEX_LOCATION=us-central1

# Optional: Disable prompt caching if needed (0 = Enabled, 1 = Disabled)
export DISABLE_PROMPT_CACHING=0
##### END Google Cloud VertexAI API for Claude Code Setup #####
Apply Changes
After saving the file, reload your terminal:

Bash
source ~/.zshrc  # or source ~/.bashrc
4. Verification
To verify that the alias is working correctly, run:

Bash
type claude
The output should confirm that claude is now an alias pointing to your service account key. You can now run claude without being prompted for gcloud re-authentication.