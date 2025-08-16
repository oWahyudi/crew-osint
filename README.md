# crew-osint
To practice running red teaming tests against an LLM model

---

## How to Test (Promptfoo + Ollama in GitHub Codespaces)

This guide shows how to spin up a Codespace, run a local LLM with Ollama, and execute Promptfoo red-team tests against it.

### Prerequisites
- A GitHub repository with Codespaces enabled
- A Linux-based Codespace (default)
- Terminal access in Codespaces

---

### 1) Open a Codespace
1. Push this repo to GitHub (e.g., branch `v1`).
2. In GitHub, click **Code → Codespaces → Create codespace** on your branch.
3. Wait for the dev container to build and open a terminal.

---

### 2) Install & Run Ollama (serves the local LLM)

```bash
# Install Ollama (Linux)
curl -fsSL https://ollama.com/install.sh | sh

# Verify installation
ollama

# Start the server (background)
ollama serve &
```

Pull and run a small model (friendly for Codespaces):

```bash
# Pull & run the model (auto-downloads on first run)
ollama run llama3.2:1b
```

You should see a chat prompt; ask a quick question to confirm it works.
> If the session disconnects, just run `ollama run llama3.2:1b` again.

---

### 3) Prepare a Promptfoo Environment

We’ll keep a clean shell using Python venv (optional but convenient) and install Promptfoo globally with npm.

```bash
# New terminal tab (Cmd/Ctrl + Shift + P → "Terminal: Create New Terminal")

# Create & activate a Python venv (optional, for a clean shell)
python -m venv vpromptfoo
source ./vpromptfoo/bin/activate

# Install Promptfoo CLI
npm install -g promptfoo
```

Initialize a baseline config (no GUI):

```bash
promptfoo redteam init --no-gui
```

This generates a `promptfooconfig.yaml` you can edit.  
> Tip: Keep initial tests small (smoke test) by lowering any `numTests` value in the config.

---

### 4) Generate Red-Team Test Prompts

```bash
# Use your config; replace the path with your actual file path
promptfoo redteam generate -c /absolute/path/to/promptfooconfig.yaml
```

This creates `redteam.yaml` with generated adversarial test cases.

---

### 5) Evaluate Against the Running Model

Make sure `ollama run llama3.2:1b` is active in another terminal, then:

```bash
# Run with single worker to avoid OOM on small machines
promptfoo redteam eval --verbose -j 1 -o results.csv
```

- `-j 1` → single-threaded execution (safer on Codespaces)
- `-o results.csv` → saves a CSV summary of the evaluation

---

### 6) View Results in the Browser

```bash
promptfoo view
```

- In Codespaces, open the **PORTS** tab, find the forwarded port for Promptfoo, and click the globe icon to open it.
- Use the UI to inspect each case and response.
- Press `Ctrl + C` in the terminal to stop the viewer.

---

### 7) Cleanup

In the Ollama terminal:

```bash
# Exit the model chat
/bye

# Stop the model if needed
ollama stop llama3.2:1b || true
```

If you started Ollama as a service in your container image (not typical in Codespaces), you can also try:

```bash
sudo systemctl stop ollama 2>/dev/null || true
```

Exit the venv:

```bash
deactivate
```

In GitHub, **Stop** the Codespace to pause billing (storage persists), or **Delete** it when done.

---

## Troubleshooting

- **Model restarts / disconnects**  
  Rerun `ollama run llama3.2:1b`.

- **Ports don’t open**  
  Ensure `promptfoo view` is running; refresh the **PORTS** tab and open the forwarded URL.

- **OOM / Slow eval**  
  Use `-j 1` and reduce total tests (e.g., lower `numTests` in your config).

- **Command not found**  
  Confirm `npm install -g promptfoo` succeeded and that you’re still in the shell where the venv is active (or reactivate with `source ./vpromptfoo/bin/activate`).

---

## Notes

- You can target different models/endpoints by editing `promptfooconfig.yaml` (e.g., HTTP API for a remote model).
- Store your generated `results.csv` in the repo (or artifacts) if you want to track regressions over time.


