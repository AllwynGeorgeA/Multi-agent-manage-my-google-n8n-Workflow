# Multi agent manage my google — n8n Workflow

## Overview
A Telegram-driven personal assistant agent that can search the web, send email, manage a Google Sheet, schedule Google Calendar events, create Google Docs, and upload files to Google Drive — all on request via chat.

## Trigger
- **Telegram Trigger** — fires on incoming Telegram messages.

## Flow
```
Telegram Trigger → AI Agent → Send a text message (Telegram)
                      ├─ OpenAI (gpt-4.1-mini)
                      ├─ Simple Memory (buffer window, keyed by chat.id)
                      ├─ Gmail (tool)
                      ├─ Google search in SerpApi (tool)
                      ├─ Upload file in Google Drive (tool)
                      ├─ Google Sheets (tool)
                      ├─ Google Calendar (tool)
                      └─ Google Docs (tool)
```

## Nodes
| Node | Type | Role |
|---|---|---|
| Telegram Trigger | telegramTrigger | Entry point, receives the Telegram message |
| AI Agent | langchain.agent | The orchestrating agent; has a detailed system prompt defining persona and tool usage |
| OpenAI | lmChatOpenAi (gpt-4.1-mini) | Language model powering the agent |
| Simple Memory | memoryBufferWindow | Per-chat memory, keyed by `telegram_{{ chat.id }}` |
| Gmail | gmailTool (`operation: send`) | Send emails |
| Google search in SerpApi | serpApiTool | General web search |
| Upload file in Google Drive | googleDriveTool | Upload files to Drive |
| Google Sheets | googleSheetsTool (`operation: create`) | Log/update data in a specific spreadsheet |
| Google Calendar | googleCalendarTool | Create calendar events |
| Google Docs | googleDocsTool | Create documents |
| Send a text message | telegram | Sends the agent's reply back to the user |

## Credentials required
- **Telegram account**
- **OpenAI account**
- **SerpApi account**
- **Gmail account** (OAuth2)
- **Google Drive account** (OAuth2)
- **Google Sheets account** (OAuth2)
- **Google Calendar account** (OAuth2)
- **Google Docs account** (OAuth2)

## System prompt
The agent is instructed to act as a "helpful, professional, and articulate AI calling agent" that confirms details before acting and maps requests to the tools above (Gmail, Google Sheet, Calendar, Google Doc).

## Notes / things to check
- This is the workflow previously fixed: the **Gmail** node had been the Human-in-the-loop variant (`gmailHitlTool`, `sendAndWait`) with no approval sub-node connected, causing every run to fail with `A Tool sub-node must be connected and enabled`. The version in this upload already uses the corrected plain `gmailTool` (`operation: send`) — good.
- **Google Sheets** node's `operation: create` will create a *new* spreadsheet on every call rather than updating the existing one referenced in `documentId` — if the intent is to log/append rows to that specific sheet, the operation should likely be `append` or `update`, not `create`.
- **Google Docs** node's `folderId` is set to an empty string (`"="`) — double check this resolves to a valid Drive location, or the doc creation may fail or land somewhere unexpected.
- No `systemMessage`-level guardrails around tool confirmation are enforced in code — the prompt asks the agent to confirm details conversationally, which is good practice but relies entirely on the model following instructions.
