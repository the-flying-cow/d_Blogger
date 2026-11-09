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

- Docker and Docker Compose
- An n8n instance
- An Ollama service with the `gemma3:latest` model(or any other model of your choice) available
- Google Sheets and Google Docs OAuth credentials with access to the configured spreadsheet and Drive folder
- SerpApi credentials
- The n8n community node used by the workflow: `n8n-nodes-serpapi`
- The n8n LangChain nodes used by the workflow

## Start n8n with Docker Compose

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
3. Configure credentials for Google Sheets, Google Docs, SerpApi, and Ollama. Imported credential references are instance-specific and may need to be remapped.
4. Set the Google Sheets document and sheet, and the Google Drive folder where generated posts should be saved.
5. Ensure the spreadsheet has columns named `Topic`, `Target Keywords`, and `Status`. Add topics with `Status` set to `Pending`; the workflow changes the selected row to `Completed` after creating its document.
6. Configure the Ollama credential's base URL so it is reachable from the n8n container. If Ollama runs on the Docker host, `localhost` inside the container refers to the container itself, not the host.
7. Test the workflow manually with a pending topic, then activate it to enable the schedule.

## Notes

- The drafting agent is instructed to use current search results and write plain-English prose. The editing agent applies the workflow's style rules and preserves technical details.
- Review generated research and drafts for accuracy before publishing.
- The Compose configuration is intended for local use: it uses HTTP and sets the n8n host and webhook URL to `localhost`. For a public or production deployment, configure an appropriate hostname, HTTPS, and secure access before exposing the service.
