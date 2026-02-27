![banner](banner.png)

# n8n Workflow Templates

A public library of production-ready [n8n](https://n8n.io/) automation templates built by [TrendAI](https://trendai.au). Each workflow is self-contained, documented, and ready to import into your own n8n instance.

**[Browse the workflow catalogue →](workflows/README.md)**

---

## How to Use

### Option 1: Import via URL

1. Open your n8n instance
2. Click **Add Workflow** > **Import from URL**
3. Paste the import link from the [workflow catalogue](workflows/README.md)

### Option 2: Import from File

1. Download `workflow.json` from any workflow folder
2. In n8n, click **Add Workflow** > **Import from File**
3. Select the downloaded file

### Option 3: Copy & Paste

1. Open a `workflow.json` file and copy the contents
2. In n8n, select all (Ctrl+A), then paste (Ctrl+V)

---

## Workflow Library

| Category | Workflows | Description |
|----------|-----------|-------------|
| [Chatbots](workflows/README.md#-chatbots) | 2 | AI agents for Facebook Messenger and other platforms |
| [CRM & Lead Capture](workflows/README.md#-crm--lead-capture) | 1 | Lead intake, HubSpot sync, contact automation |
| [Content & Media](workflows/README.md#-content--media) | 1 | YouTube summarizers, video processing, media automation |
| Email | coming soon | Sequences, outreach, and notifications |
| Social | coming soon | Content posting and engagement |
| Reporting | coming soon | KPI dashboards and automated summaries |

---

## Folder Structure

```
workflows/
  {workflow-name}/
    workflow.json   # n8n workflow (importable)
    README.md       # Setup guide, prerequisites, credentials
    preview.png     # Workflow screenshot
```

---

## Requirements

These workflows are built and tested on **self-hosted n8n Community Edition v2.0.2**. Most should work on n8n Cloud and other recent versions.

---

## About TrendAI

TrendAI builds AI automation systems for Australian SMBs. These workflows are published as free resources for the n8n community.

🌐 [trendai.au](https://trendai.au) · 📂 [GitHub](https://github.com/trendai-au-lab)

---

## License

MIT
