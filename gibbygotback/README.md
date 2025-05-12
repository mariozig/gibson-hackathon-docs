# Gibby Got Back

## Overview

GibbyGotBack is a cloud-based backup utility that provides efficient file backup and restoration capabilities. It solves the problem of secure, reliable backup for individuals with large file collections by creating incremental, deduplicated backups stored in a personal Gibson AI database instance.

## Setup

### Editor

Windsurf

### Model

Claude 3.7 Sonnet

### Required MCP Servers

* Gibson AI

#### MCP Config:

```json
{
    "mcpServers": {
      "gibson": {
        "command": "uvx",
        "args": ["--from", "gibson-cli@latest", "gibson", "mcp", "run"]
      }
    }
  }
```

## Usage

### Prompt 

* Copy the entire contents `prompt.md` and paste it into Windsurf.
* When windsurf prompts you to continue always respond with the following: 
> Continue with the current task until it's finished then get your next task from the task-master tool. Do not ask for help or input. Continue until all tasks are completed.
