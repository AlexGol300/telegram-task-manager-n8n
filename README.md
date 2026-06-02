# Telegram Task Manager on n8n

[Русская версия](./README_RU.md)

AI-powered Telegram task manager built with n8n, Telegram Bot API, and OpenAI.
The workflow creates tasks from natural language, extracts deadlines, stores tasks in an n8n Data Table, supports task updates, and sends reminders 30 minutes before the deadline.

## Overview

This project is an n8n workflow for a Telegram bot that supports four core user commands: `/start`, `/newtask`, `/list`, and `/updatetask`.
The workflow includes one branch for command routing, one branch for task creation, one branch for task updates, and one scheduled branch for deadline reminders.

![Workflow Schema](docs/workflow-schema-original.jpg)

## Features

- Create tasks from natural language messages
- Extract deadlines from explicit and relative date expressions
- Validate deadlines before saving tasks
- View all active tasks in Telegram
- Update deadline, mark a task as completed, or delete a task
- Send automatic reminders 30 minutes before deadline
- Store tasks in an n8n Data Table

## Demo commands

```text
/newtask Call the workshop manager tomorrow at 19:30
/list
/updatetask TASK-12 postpone to Friday 11:00
/updatetask TASK-12 done
/updatetask TASK-12 delete
```

## Command reference

| Command | Purpose | Result |
|---|---|---|
| `/start` | Start the bot | Sends welcome message and command list |
| `/newtask` | Create a new task | Extracts task text and deadline, then saves the task |
| `/list` | Show active tasks | Returns the current task list for the user |
| `/updatetask` | Change existing task | Updates deadline, completes task, or removes task |

## Architecture

### 1. Input and routing
- **Telegram Trigger** receives incoming messages.
- **Config** extracts `command`, `message_content`, and `chat_id`.
- **Switch** routes the execution based on the command.

### 2. Task creation
- **Information Extractor** parses `task_content` and `deadline_at`.
- **If** checks whether the deadline is valid.
- **Insert row** stores the task in the `Tasks` Data Table.
- Telegram node sends a success or validation message.

### 3. Task listing
- **Get row(s)** loads active tasks for the current `chat_id`.
- **Date & Time**, **Aggregate**, and **Edit Fields** prepare a readable message.
- Telegram node sends the formatted task list.

### 4. Task update flow
- A second **Information Extractor** parses `task_id`, `action`, and optionally a new `deadline_at`.
- **Switch1** routes the action to deadline update, completion, or deletion.
- Data Table nodes update or remove the relevant task.

### 5. Reminder flow
- **Schedule Trigger** runs every minute.
- **Get row(s)1** loads unfinished and not-yet-notified tasks.
- **Edit Fields1** checks whether the reminder should be sent.
- Telegram node sends the reminder, then **Update row(s)** marks the task state accordingly.

## Data model

The workflow uses a Data Table named `Tasks`.

| Field | Type | Description |
|---|---|---|
| `chat_id` | number | Telegram chat ID |
| `TaskContent` | string | User task text |
| `DeadlineAt` | dateTime | Task deadline |
| `Finished` | boolean | Completion flag |
| `notified` | boolean | Reminder sent flag |

## Additional files

- [Command examples (EN)](./examples/telegram-commands.md)
- [Command examples (RU)](./examples/telegram-commands-ru.md)
- [Prompt for task creation extractor (RU)](./prompts/create-task-extractor.txt)
- [Prompt for task creation extractor (EN)](./prompts/create-task-extractor-en.txt)
- [Prompt for task update extractor (RU)](./prompts/update-task-extractor.txt)
- [Prompt for task update extractor (EN)](./prompts/update-task-extractor-en.txt)

## Project structure

```text
telegram-task-manager-n8n/
├─ README.md
├─ README_RU.md
├─ LICENSE
├─ .gitignore
├─ docs/
│  ├─ workflow-schema-original.jpg
│  └─ screenshots/
├─ workflows/
│  └─ telegram-task-manager.json
├─ prompts/
│  ├─ create-task-extractor.txt
│  ├─ create-task-extractor-en.txt
│  ├─ update-task-extractor.txt
│  └─ update-task-extractor-en.txt
└─ examples/
   ├─ telegram-commands.md
   └─ telegram-commands-ru.md
```

## Quick start

1. Install and launch n8n.
2. Create a Telegram bot with [@BotFather](https://t.me/BotFather).
3. Add Telegram credentials in n8n.
4. Add OpenAI credentials in n8n.
5. Create a Data Table named `Tasks` with the required fields.
6. Import `workflows/telegram-task-manager.json` into n8n.
7. Reconnect all Telegram and OpenAI nodes to your own credentials.
8. Update every Data Table node so it points to your `Tasks` table.
9. Activate the workflow.

## Setup details

### Telegram
- Create a bot token in BotFather.
- Add the token to the `Telegram account` credential in n8n.
- Ensure the workflow is active so Telegram webhooks can work correctly.

### OpenAI
- Create an OpenAI API key.
- Add it to the `OpenAI account` credential in n8n.
- Verify model availability in your workspace before enabling the workflow.

### Data Table
Create a Data Table named `Tasks` with these columns:

```text
chat_id      number
TaskContent  string
DeadlineAt   dateTime
Finished     boolean
notified     boolean
```

## Import checklist

Before turning the workflow on, verify the following:

- Telegram credentials are reconnected
- OpenAI credentials are reconnected
- All Data Table nodes use your own `Tasks` table
- The schedule trigger interval is correct for your use case
- The bot replies correctly to `/start`
- Test task creation with a real message like `/newtask Submit report tomorrow 18:00`

## Troubleshooting

### Tasks are not saved
- Check the `Insert row` node configuration.
- Verify the `Tasks` Data Table schema.
- Confirm that `deadline_at` is parsed correctly by the extractor node.

### Bot does not respond
- Verify Telegram credentials.
- Make sure the workflow is active.
- Check whether the Telegram Trigger webhook is registered.

### Reminders are not sent
- Check the `Schedule Trigger` status.
- Verify that `Finished = false` and `notified = false` for test rows.
- Confirm that deadline comparison logic matches your timezone.# Telegram Task Manager on n8n

[Русская версия](./README_RU.md)

AI-powered Telegram task manager built with n8n, Telegram Bot API, and OpenAI.
The workflow creates tasks from natural language, extracts deadlines, stores tasks in an n8n Data Table, supports task updates, and sends reminders 30 minutes before the deadline.

## Overview

This project is an n8n workflow for a Telegram bot that supports four core user commands: `/start`, `/newtask`, `/list`, and `/updatetask`.
The workflow includes one branch for command routing, one branch for task creation, one branch for task updates, and one scheduled branch for deadline reminders.

![Workflow Schema](docs/workflow-schema-original.jpg)

## Features

- Create tasks from natural language messages
- Extract deadlines from explicit and relative date expressions
- Validate deadlines before saving tasks
- View all active tasks in Telegram
- Update deadline, mark a task as completed, or delete a task
- Send automatic reminders 30 minutes before deadline
- Store tasks in an n8n Data Table

## Demo commands

```text
/newtask Call the workshop manager tomorrow at 19:30
/list
/updatetask TASK-12 postpone to Friday 11:00
/updatetask TASK-12 done
/updatetask TASK-12 delete
```

## Command reference

| Command | Purpose | Result |
|---|---|---|
| `/start` | Start the bot | Sends welcome message and command list |
| `/newtask` | Create a new task | Extracts task text and deadline, then saves the task |
| `/list` | Show active tasks | Returns the current task list for the user |
| `/updatetask` | Change existing task | Updates deadline, completes task, or removes task |

## Architecture

### 1. Input and routing
- **Telegram Trigger** receives incoming messages.
- **Config** extracts `command`, `message_content`, and `chat_id`.
- **Switch** routes the execution based on the command.

### 2. Task creation
- **Information Extractor** parses `task_content` and `deadline_at`.
- **If** checks whether the deadline is valid.
- **Insert row** stores the task in the `Tasks` Data Table.
- Telegram node sends a success or validation message.

### 3. Task listing
- **Get row(s)** loads active tasks for the current `chat_id`.
- **Date & Time**, **Aggregate**, and **Edit Fields** prepare a readable message.
- Telegram node sends the formatted task list.

### 4. Task update flow
- A second **Information Extractor** parses `task_id`, `action`, and optionally a new `deadline_at`.
- **Switch1** routes the action to deadline update, completion, or deletion.
- Data Table nodes update or remove the relevant task.

### 5. Reminder flow
- **Schedule Trigger** runs every minute.
- **Get row(s)1** loads unfinished and not-yet-notified tasks.
- **Edit Fields1** checks whether the reminder should be sent.
- Telegram node sends the reminder, then **Update row(s)** marks the task state accordingly.

## Data model

The workflow uses a Data Table named `Tasks`.

| Field | Type | Description |
|---|---|---|
| `chat_id` | number | Telegram chat ID |
| `TaskContent` | string | User task text |
| `DeadlineAt` | dateTime | Task deadline |
| `Finished` | boolean | Completion flag |
| `notified` | boolean | Reminder sent flag |

## Additional files

- [Command examples (EN)](./examples/telegram-commands.md)
- [Command examples (RU)](./examples/telegram-commands-ru.md)
- [Prompt for task creation extractor (RU)](./prompts/create-task-extractor.txt)
- [Prompt for task creation extractor (EN)](./prompts/create-task-extractor-en.txt)
- [Prompt for task update extractor (RU)](./prompts/update-task-extractor.txt)
- [Prompt for task update extractor (EN)](./prompts/update-task-extractor-en.txt)

## Project structure

```text
telegram-task-manager-n8n/
├─ README.md
├─ README_RU.md
├─ LICENSE
├─ .gitignore
├─ docs/
│  ├─ workflow-schema-original.jpg
│  └─ screenshots/
├─ workflows/
│  └─ telegram-task-manager.json
├─ prompts/
│  ├─ create-task-extractor.txt
│  ├─ create-task-extractor-en.txt
│  ├─ update-task-extractor.txt
│  └─ update-task-extractor-en.txt
└─ examples/
   ├─ telegram-commands.md
   └─ telegram-commands-ru.md
```

## Quick start

1. Install and launch n8n.
2. Create a Telegram bot with [@BotFather](https://t.me/BotFather).
3. Add Telegram credentials in n8n.
4. Add OpenAI credentials in n8n.
5. Create a Data Table named `Tasks` with the required fields.
6. Import `workflows/telegram-task-manager.json` into n8n.
7. Reconnect all Telegram and OpenAI nodes to your own credentials.
8. Update every Data Table node so it points to your `Tasks` table.
9. Activate the workflow.

## Setup details

### Telegram
- Create a bot token in BotFather.
- Add the token to the `Telegram account` credential in n8n.
- Ensure the workflow is active so Telegram webhooks can work correctly.

### OpenAI
- Create an OpenAI API key.
- Add it to the `OpenAI account` credential in n8n.
- Verify model availability in your workspace before enabling the workflow.

### Data Table
Create a Data Table named `Tasks` with these columns:

```text
chat_id      number
TaskContent  string
DeadlineAt   dateTime
Finished     boolean
notified     boolean
```

## Import checklist

Before turning the workflow on, verify the following:

- Telegram credentials are reconnected
- OpenAI credentials are reconnected
- All Data Table nodes use your own `Tasks` table
- The schedule trigger interval is correct for your use case
- The bot replies correctly to `/start`
- Test task creation with a real message like `/newtask Submit report tomorrow 18:00`

## Troubleshooting

### Tasks are not saved
- Check the `Insert row` node configuration.
- Verify the `Tasks` Data Table schema.
- Confirm that `deadline_at` is parsed correctly by the extractor node.

### Bot does not respond
- Verify Telegram credentials.
- Make sure the workflow is active.
- Check whether the Telegram Trigger webhook is registered.

### Reminders are not sent
- Check the `Schedule Trigger` status.
- Verify that `Finished = false` and `notified = false` for test rows.
- Confirm that deadline comparison logic matches your timezone.

## Security notes

- Do not commit real API keys or Telegram bot tokens.
- Do not publish production chat IDs or personal data.
- Export the workflow without secrets before uploading to GitHub.

## Notes

- The workflow uses OpenAI extraction nodes for both task creation and task update logic.
- Reminder logic is implemented as a separate scheduled branch.
- After importing the workflow, environment-specific IDs must be reconfigured.# Telegram Task Manager on n8n

[Русская версия](./README_RU.md)

AI-powered Telegram task manager built with n8n, Telegram Bot API, and OpenAI.
The workflow creates tasks from natural language, extracts deadlines, stores tasks in an n8n Data Table, supports task updates, and sends reminders 30 minutes before the deadline.

## Overview

This project is an n8n workflow for a Telegram bot that supports four core user commands: `/start`, `/newtask`, `/list`, and `/updatetask`.
The workflow includes one branch for command routing, one branch for task creation, one branch for task updates, and one scheduled branch for deadline reminders.

![Workflow Schema](docs/workflow-schema-original.jpg)

## Features

- Create tasks from natural language messages
- Extract deadlines from explicit and relative date expressions
- Validate deadlines before saving tasks
- View all active tasks in Telegram
- Update deadline, mark a task as completed, or delete a task
- Send automatic reminders 30 minutes before deadline
- Store tasks in an n8n Data Table

## Demo commands

```text
/newtask Call the workshop manager tomorrow at 19:30
/list
/updatetask TASK-12 postpone to Friday 11:00
/updatetask TASK-12 done
/updatetask TASK-12 delete
```

## Command reference

| Command | Purpose | Result |
|---|---|---|
| `/start` | Start the bot | Sends welcome message and command list |
| `/newtask` | Create a new task | Extracts task text and deadline, then saves the task |
| `/list` | Show active tasks | Returns the current task list for the user |
| `/updatetask` | Change existing task | Updates deadline, completes task, or removes task |

## Architecture

### 1. Input and routing
- **Telegram Trigger** receives incoming messages.
- **Config** extracts `command`, `message_content`, and `chat_id`.
- **Switch** routes the execution based on the command.

### 2. Task creation
- **Information Extractor** parses `task_content` and `deadline_at`.
- **If** checks whether the deadline is valid.
- **Insert row** stores the task in the `Tasks` Data Table.
- Telegram node sends a success or validation message.

### 3. Task listing
- **Get row(s)** loads active tasks for the current `chat_id`.
- **Date & Time**, **Aggregate**, and **Edit Fields** prepare a readable message.
- Telegram node sends the formatted task list.

### 4. Task update flow
- A second **Information Extractor** parses `task_id`, `action`, and optionally a new `deadline_at`.
- **Switch1** routes the action to deadline update, completion, or deletion.
- Data Table nodes update or remove the relevant task.

### 5. Reminder flow
- **Schedule Trigger** runs every minute.
- **Get row(s)1** loads unfinished and not-yet-notified tasks.
- **Edit Fields1** checks whether the reminder should be sent.
- Telegram node sends the reminder, then **Update row(s)** marks the task state accordingly.

## Data model

The workflow uses a Data Table named `Tasks`.

| Field | Type | Description |
|---|---|---|
| `chat_id` | number | Telegram chat ID |
| `TaskContent` | string | User task text |
| `DeadlineAt` | dateTime | Task deadline |
| `Finished` | boolean | Completion flag |
| `notified` | boolean | Reminder sent flag |

## Additional files

- [Command examples (EN)](./examples/telegram-commands.md)
- [Command examples (RU)](./examples/telegram-commands-ru.md)
- [Prompt for task creation extractor (RU)](./prompts/create-task-extractor.txt)
- [Prompt for task creation extractor (EN)](./prompts/create-task-extractor-en.txt)
- [Prompt for task update extractor (RU)](./prompts/update-task-extractor.txt)
- [Prompt for task update extractor (EN)](./prompts/update-task-extractor-en.txt)

## Project structure

```text
telegram-task-manager-n8n/
├─ README.md
├─ README_RU.md
├─ LICENSE
├─ .gitignore
├─ docs/
│  ├─ workflow-schema-original.jpg
│  └─ screenshots/
├─ workflows/
│  └─ telegram-task-manager.json
├─ prompts/
│  ├─ create-task-extractor.txt
│  ├─ create-task-extractor-en.txt
│  ├─ update-task-extractor.txt
│  └─ update-task-extractor-en.txt
└─ examples/
   ├─ telegram-commands.md
   └─ telegram-commands-ru.md
```

## Quick start

1. Install and launch n8n.
2. Create a Telegram bot with [@BotFather](https://t.me/BotFather).
3. Add Telegram credentials in n8n.
4. Add OpenAI credentials in n8n.
5. Create a Data Table named `Tasks` with the required fields.
6. Import `workflows/telegram-task-manager.json` into n8n.
7. Reconnect all Telegram and OpenAI nodes to your own credentials.
8. Update every Data Table node so it points to your `Tasks` table.
9. Activate the workflow.

## Setup details

### Telegram
- Create a bot token in BotFather.
- Add the token to the `Telegram account` credential in n8n.
- Ensure the workflow is active so Telegram webhooks can work correctly.

### OpenAI
- Create an OpenAI API key.
- Add it to the `OpenAI account` credential in n8n.
- Verify model availability in your workspace before enabling the workflow.

### Data Table
Create a Data Table named `Tasks` with these columns:

```text
chat_id      number
TaskContent  string
DeadlineAt   dateTime
Finished     boolean
notified     boolean
```

## Import checklist

Before turning the workflow on, verify the following:

- Telegram credentials are reconnected
- OpenAI credentials are reconnected
- All Data Table nodes use your own `Tasks` table
- The schedule trigger interval is correct for your use case
- The bot replies correctly to `/start`
- Test task creation with a real message like `/newtask Submit report tomorrow 18:00`

## Troubleshooting

### Tasks are not saved
- Check the `Insert row` node configuration.
- Verify the `Tasks` Data Table schema.
- Confirm that `deadline_at` is parsed correctly by the extractor node.

### Bot does not respond
- Verify Telegram credentials.
- Make sure the workflow is active.
- Check whether the Telegram Trigger webhook is registered.

### Reminders are not sent
- Check the `Schedule Trigger` status.
- Verify that `Finished = false` and `notified = false` for test rows.
- Confirm that deadline comparison logic matches your timezone.

## Security notes

- Do not commit real API keys or Telegram bot tokens.
- Do not publish production chat IDs or personal data.
- Export the workflow without secrets before uploading to GitHub.

## Notes

- The workflow uses OpenAI extraction nodes for both task creation and task update logic.
- Reminder logic is implemented as a separate scheduled branch.
- After importing the workflow, environment-specific IDs must be reconfigured.

## Security notes

- Do not commit real API keys or Telegram bot tokens.
- Do not publish production chat IDs or personal data.
- Export the workflow without secrets before uploading to GitHub.

## Notes

- The workflow uses OpenAI extraction nodes for both task creation and task update logic.
- Reminder logic is implemented as a separate scheduled branch.
- After importing the workflow, environment-specific IDs must be reconfigured.