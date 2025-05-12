# Local Story Vault

## Overview

Local Story Vault is a location-based audio storytelling platform that allows residents to record, geo-tag, and share short audio stories about their neighborhood. The platform transforms ordinary streets into galleries of human stories, preserving local heritage and cultural memory while creating immersive experiences for both residents and visitors.

## ❗️WARNING ❗️

When I ran this prompt it took close to 9 hours to complete. 🐢🐢🐢

## Setup

### Editor

#### Windsurf

* Add this project's `.windsurfrules` file to your project root.
* Delete all saved windsurf memories

### Model

Gemini 2.5 Pro

### API Keys

You will need API keys for the following: 

* Perplexity (for Task Master)
* Anthropic (for Task Master)
* OpenAI (for usage of the Whisper API audio -> text)
* Google Maps (for plotting points on a map)

**Note**: These will ultimately be use as environment variables but the model creates .env files differently every time. 

### Required MCP Servers

* Gibson AI
* [Task Master AI](https://github.com/eyaltoledano/claude-task-master)

##### MCP Config:

```json
{
    "mcpServers": {
      "taskmaster-ai": {
        "command": "npx",
        "args": ["-y", "--package=task-master-ai", "task-master-ai"],
        "env": {
          "ANTHROPIC_API_KEY": "YOUR_ANTHROPIC_API_KEY",
          "PERPLEXITY_API_KEY": "YOUR_PERPLEXITY_API_KEY",
          "MODEL": "claude-3-7-sonnet-20250219",
          "PERPLEXITY_MODEL": "sonar-pro",
          "MAX_TOKENS": "64000",
          "TEMPERATURE": "0.2",
          "DEFAULT_SUBTASKS": "5",
          "DEFAULT_PRIORITY": "medium"
        }
      },  
      "gibson": {
        "command": "uvx",
        "args": ["--from", "gibson-cli@latest", "gibson", "mcp", "run"]
      }
    }
  }
```

## Usage

### Repo
This prompt assumes you will be runnning it from within [Gibson's Next.js starter template](https://github.com/GibsonAI/next-app).

The easiest way to set that up is by running this command in your terminal:

```bash
bash <(curl -s https://raw.githubusercontent.com/GibsonAI/next-app/main/setup.sh)
```



### Prompt 

* Copy the entire contents `prompt.md` and paste it into Windsurf.
* When windsurf prompts you to continue always respond with the following: 
> Continue with the current task until it's finished then get your next task from the task-master tool. Do not ask for help or input. Continue until all tasks are completed.
