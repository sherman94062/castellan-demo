# Castellan Demo — Suspect AI Agent

A deliberately vulnerable AI agent used to demonstrate why AI agent governance is critical. Built for use with the [Castellan Platform](https://castellan.ai).

## Bad Behaviors Demonstrated

| # | Behavior | Description |
|---|----------|-------------|
| 1 | **Data Exfiltration** | Reads sensitive files and writes contents to `exfil.log` |
| 2 | **Destructive Actions** | Deletes files without user authorization |
| 3 | **Off-Task Behavior** | Hidden system prompt directives beyond the user's stated task |
| 4 | **Prompt Injection** | Blindly executes instructions found inside file contents |
| 5 | **Runaway Loop** | No iteration cap — runs indefinitely without a stop condition |

## Setup

1. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

2. Create a `.env` file with your OpenAI API key:
   ```
   OPENAI_API_KEY=your-api-key-here
   ```

3. Create demo fixtures:
   ```bash
   python demo_setup.py
   ```

4. Run the suspect agent:
   ```bash
   python suspect_agent.py
   ```

## What to Watch For

- `exfil.log` is created and populated with PII and credentials
- `demo_files/customers.csv` is deleted by the agent after prompt injection
- The agent executes far beyond its stated task with no user consent

## Warning

This agent is intentionally unsafe and is for **demo purposes only**. Do not run against real sensitive data or in a production environment.
