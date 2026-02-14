![banner](banner.png)

# n8n Workflow Templates

A public library of production-ready [n8n](https://n8n.io/) automation templates. Each workflow is self-contained, documented, and ready to import into your own n8n instance.

**[Browse all workflows](https://github.com/trendai-au-lab/n8n-workflow/blob/main/workflows/README.md)**

## How to Use

### Option 1: Import via URL

1. Open your n8n instance
2. Click **Add Workflow** > **Import from URL**
3. Paste the import link from the [workflow catalogue](https://github.com/trendai-au-lab/n8n-workflow/blob/main/workflows/README.md)

### Option 2: Import from File

1. Download `workflow.json` from any workflow folder
2. In n8n, click **Add Workflow** > **Import from File**
3. Select the downloaded file

### Option 3: Copy & Paste

1. Open a `workflow.json` file and copy the contents
2. In n8n, select all (Ctrl+A), then paste (Ctrl+V)

## Folder Structure

```
workflows/
  {workflow-name}/
    workflow.json   # n8n workflow (importable)
    README.md       # Setup guide, prerequisites, credentials
    preview.png     # Workflow screenshot (optional)
```

Each workflow README contains:
- **Prerequisites** — required accounts and services
- **Credentials table** — which nodes need which credentials
- **Step-by-step setup guide** — from credential creation to testing

## Requirements

These workflows are built and tested on **self-hosted n8n Community Edition v2.0.2**. Most should work on n8n Cloud and other recent versions.

## Contributing

1. Fork this repository
2. Create a workflow folder under `workflows/` following the structure above
3. Include `workflow.json` and `README.md`
4. Ensure no personal data (API keys, tokens, emails) is in the workflow JSON — use placeholders like `REPLACE_WITH_YOUR_CREDENTIAL_ID`
5. Submit a Pull Request

## License

MIT
