# CrewAI OSINT Lab (GitHub Codespaces)
This guide walks you through setting up a GitHub Codespace, running a local LLM with Ollama, and using a Jupyter Notebook–based CrewAI workflow to gather OSINT about a company.

---

## Summary (What you’ll do)
- Reuse or create a GitHub Codespace
- Ensure Ollama and a small local model are running (`llama3.2:1b`)
- Create a Python virtual environment and install Jupyter
- Configure Jupyter for remote access inside Codespaces
- Upload the provided `.ipynb` notebook, set your EXA API key, and run the notebook
- Inspect the generated `osint_report.md`

> Python requirement for CrewAI in this lab: **Python >= 3.10 and < 3.13**.

---

## Prerequisites
- A GitHub repository with **Codespaces** enabled
- Basic terminal usage in VS Code Codespaces

---

## 1) Open or Reuse a Codespace
- If you previously **stopped** (not deleted) a Codespace, you can **reopen** it and skip setup steps you’ve already done.
- Otherwise, create a new Codespace: **Code → Codespaces → Create codespace** on your desired branch.

> Tip: Use the VS Code **Terminal** pane for commands. You can open a new terminal via **Ctrl/Cmd + Shift + P → “Terminal: Create New Terminal.”**

---

## 2) Install & Run Ollama (Local LLM)
Skip if Ollama is already installed and running.

```bash
# Install Ollama (Linux)
curl -fsSL https://ollama.com/install.sh | sh

# Verify installation
ollama

# Start the server (background)
ollama serve &

# Pull & run a small model suitable for Codespaces
ollama run llama3.2:1b
```

- You should see a chat prompt. Send a quick message to confirm it works.
- If the session disconnects, re-run `ollama run llama3.2:1b`.

---

## 3) Create a Python Virtual Environment & Install Jupyter
In a **new terminal** (optional but recommended for clarity):

```bash
# Create and activate a Python venv
python -m venv vcrewai
cd vcrewai
source ./bin/activate

# Confirm Python version (should be >=3.10 and <3.13 for this lab)
python --version

# Install Jupyter Notebook
pip install jupyter

# Check installation
jupyter-notebook --version
```

### Generate and Edit Jupyter Config
```bash
# Create the config (typically at ~/.jupyter/jupyter_notebook_config.py)
jupyter-notebook --generate-config

# Open the file in nano
nano ~/.jupyter/jupyter_notebook_config.py
```
Add the following **two lines** right after the line:
`c = get_config()  #noqa`

```python
c.ServerApp.open_browser = False      # Prevent opening a browser on the server
c.ServerApp.allow_remote_access = True  # Allow external access
```

Save & exit nano: **Ctrl + X**, then **Y**, then **Enter**.

---

## 4) Prepare the Notebook & EXA API Key
1. **Download** the supplied Jupyter Notebook (`.ipynb`) from Canvas.
2. **Upload** it to the `vcrewai` directory in your Codespace.
3. In a browser, go to **exa.ai**, sign in with Google, and create a new API key named `crewai`.
4. In the terminal, create a `.env` file in `vcrewai` with your key:
   ```bash
   echo "EXA_API_KEY=YOUR_EXA_API_KEY_HERE" > .env
   ```

---

## 5) Start Jupyter & Log In
```bash
jupyter notebook
```
- A URL will be printed in the terminal that includes a `token=...` value.
- **Ctrl/Cmd + Click** the URL to open Jupyter’s login page.
- Copy the **token** from the terminal and paste it into the login page.

---

## 6) Run the Notebook
- In Jupyter, make a **copy** of the uploaded `.ipynb` and work on the copy.
- **Open** the copy, then change all `verbose=True` to `verbose=False` in the notebook.
- In the **Ollama** terminal, ensure the model is active by chatting with it.
- Back in Jupyter, **run cells** one by one (**Shift + Enter**).

Execution markers:
- `[*]` means the cell is running.
- `[n]` (e.g., `[1]`, `[2]`) means completed; fresh kernels usually start at `[1]`.

Prompts & output:
- When prompted, enter a **company name** to gather OSINT about (try a **single-word** company name first).
- If you hit a **recursion error (max depth exceeded)**, ensure **all** occurrences of `verbose` are set to `False` and rerun.
- The notebook will generate an **`osint_report.md`** in your workspace.

---

## 7) Rerun If Needed
If you are not satisfied with the report, rerun the crew (e.g., via a notebook cell calling):
```python
crew.kickoff()
```

---

## 8) Shutdown & Cleanup
- In Jupyter, choose **File → Shut Down** to stop the server.
- If you no longer need the environment, **Stop** or **Delete** your Codespace in GitHub.

---

## Tips
- Locate Jupyter:
  ```bash
  which jupyter-notebook
  ```

- If you encounter **sqlite errors** when running Python code inside the notebook:
  ```python
  # In a notebook cell:
  !pip install pysqlite3-binary

  __import__('pysqlite3')
  import sys
  sys.modules['sqlite3'] = sys.modules.pop('pysqlite3')
  ```

- Ollama quick recap:
  ```bash
  # Install
  curl -fsSL https://ollama.com/install.sh | sh

  # Run (server + model)
  ollama
  ollama serve &
  ollama run llama3.2:1b

  # Stop model / server
  ollama stop llama3.2:1b
  sudo service ollama stop || sudo systemctl stop ollama
  ```

---

## Reference
- Installing Jupyter Notebook on Ubuntu 24.04: https://docs.vultr.com/how-to-install-jupyter-notebook-on-ubuntu-24-04
