# d_Blogger

An n8n workflow that turns queued blog topics into researched, edited Google Docs. It uses Ollama for drafting and editing, SerpApi for search, and Google Sheets to manage the topic queue.

## How it works

The workflow in [`workflow.json`](./workflow.json) is scheduled for Tuesdays and Saturdays at 10:00 in the `Asia/Kolkata` time zone. On each run, it:

1. Finds the first row in the configured Google Sheets document whose `Status` is `Pending`.
2. Searches the web with SerpApi and passes the topic, target keywords, and search results to an AI drafting agent.
3. Uses an Ollama model (`gemma3:latest`) to draft the post, then a second Ollama agent to edit it for a natural, practical engineering tone.
4. Creates a Google Doc titled `Blog - <topic>` in the configured Drive folder and inserts the edited post.
5. Updates the matching spreadsheet row to `Completed`.

The workflow processes one pending topic per scheduled run. The workflow starts inactive; activate it in n8n when you are ready to run it.

## Requirements

- Docker Desktop (Windows/macOS) or Docker Engine with the Docker Compose v2 plugin (Linux); `docker compose` must be available
- Self-hosted n8n Community Edition, which the included Compose file starts for you (no separate n8n installation or n8n Cloud subscription is required)
- An Ollama service with the `gemma3:latest` model (or another model of your choice) available
- Google Sheets and Google Docs OAuth credentials
- SerpApi credentials
- The n8n community node used by the workflow: `n8n-nodes-serpapi`
- The n8n LangChain nodes used by the workflow

## Start n8n Community Edition with Docker Compose

From this directory, run:

```powershell
docker compose up -d
```

Open [http://localhost:5678](http://localhost:5678) and complete the initial n8n setup. The Compose file publishes port `5678` and persists n8n data in the `n8n_data` Docker volume.

To stop the container while retaining its data:

```powershell
docker compose down
```

## Configure the workflow

1. In n8n, install the required community node (`n8n-nodes-serpapi`) and make sure the LangChain nodes are available.
2. Import `workflow.json`.
3. Assign credentials to the workflow nodes: Google Sheets OAuth2 on both Sheets nodes, Google Docs OAuth2 on both Docs nodes, your SerpApi credential on the search node, and your Ollama credential on both Ollama model nodes. Credentials are not included in this repository.
4. In both Google Sheets nodes, select your spreadsheet and sheet. The workflow contains `YOUR_GOOGLE_SHEETS_DOCUMENT_ID`, `YOUR_SPREADSHEET_NAME`, `YOUR_SHEET_ID`, and `YOUR_SHEET_NAME` placeholders; select the resources in n8n to replace them.
5. In the **create doc** node, select the Drive folder for generated posts. Replace `YOUR_GOOGLE_DRIVE_FOLDER_ID` with your folder by selecting it in n8n.
6. Ensure the spreadsheet has columns named `Topic`, `Target Keywords`, and `Status`. Add topics with `Status` set to `Pending`; the workflow changes the selected row to `Completed` after creating its document.
7. Configure the Ollama credential's base URL so it is reachable from the n8n container. If Ollama runs on the Docker host, `localhost` inside the container refers to the container itself, not the host.
8. Test the workflow manually with a pending topic, then activate it to enable the schedule.

## Notes

- The drafting agent is instructed to use current search results and write plain-English prose. The editing agent applies the workflow's style rules and preserves technical details.
- Review generated research and drafts for accuracy before publishing.
- The Compose configuration is intended for local use: it uses HTTP and sets the n8n host and webhook URL to `localhost`. For a public or production deployment, configure an appropriate hostname, HTTPS, and secure access before exposing the service.

## License

This project is licensed under the MIT License. See [`LICENSE`](./LICENSE) for details.
