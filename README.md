markdown
# 🤖 Autonomous AI Agent CTF Harness

An advanced, token-efficient **Autonomous Cyber Security Agent Harness** designed to solve Capture The Flag (CTF) challenges entirely on its own. 

This system bridges a high-reasoning Large Language Model (**Qwen 27B** via Groq's low-latency LPU engine) with an isolated, privilege-escalated **Kali Linux Docker Sandbox container**. It creates an automated **Observation → Thought → Action → Execution** feedback loop, allowing the agent to dynamically probe networks, exploit targets, write custom scripts, parse raw file headers, and intercept flags completely hands-free.

---

## 🛠️ How It Works (The Feedback Loop)

Use code with caution.
[ Your Host Machine ]
│
▼ (Passes challenge description text dynamically)
[ Python Orchestrator ] <───(Low-Latency API)───> [ Groq LPU Engine (Qwen 27B) ]
│                                                    │
▼ (Lifts network / socket restrictions)              ▼
[ Privileged Docker Container Sandbox ] ◄───────── (Issues XML tags: ...)
│
▼ (Shared Host Networking Adapter Interface)
[ Target Challenge Server (Live Port/URL) ]

1. **Interactive Prompt Intake:** The script boots and allows you to copy-paste an entire multi-line CTF challenge description directly into the terminal, concluding with a simple `DONE` signal.
2. **Cognitive Loop Initializer:** The orchestrator packages your challenge instructions with a precise domain-rules system prompt and ships it to the LLM.
3. **Structured Command Interception:** The model reasons through its next step and emits an execution payload wrapped within structured text tags (e.g., `<cmd>nmap -sV target</cmd>`).
4. **Sandboxed Command Processing:** The Python framework catches the tag via regex, strips the text, and processes it inside a standalone, headless **Kali Linux Docker Container**. By running through `bash -c`, it natively supports piping (`|`), chaining (`&&`, `;`), and text outputs (`>`, `>>`).
5. **Dynamic Failure Recovery:** If a tool is missing or a command errors out (e.g., `xxd: command not found`), the container passes the error back to the LLM. The agent intelligently reads the error message and crafts a programmatic alternative (e.g., swapping to a pure Python hex conversion script) on the fly.
6. **Context Shielding & Truncation:** If a command dumps a massive screen log, the framework automatically truncates the middle to protect your API context limit window.
7. **Instant Flag Interception:** Every block of text coming out of the container terminal is instantly evaluated by a broad-spectrum regex filter watching for common flags (`flag{`, `HTB{`, `picoCTF{`, `thm{`, or raw hex hashes). The microsecond a flag hits the terminal buffer, the script halts execution, saves your tokens, prints a success banner, and clean-destroys the sandbox environment.

---

## 📦 Prerequisites

Ensure you have your core dependencies operational on your host machine before building the harness:

* **OS:** Kali Linux, Ubuntu, or any standard Linux distribution with Docker installed.
* **Docker Daemon:** Must be running locally (`systemctl start docker`).
* **Python Engine:** Version 3.10+ running within an active virtual environment.
* **API Access:** A valid, free developer account API token from [Groq Console](https://groq.com).

---

## 🚀 Step-by-Step Setup Guide

### 1. Configure the Local Docker Sandbox Workspace
Create a folder for your agent project and navigate inside it:
```bash
mkdir -p ~/agent_harness
cd ~/agent_harness
```

Create a file named `Dockerfile` (no file extension) to compile an automated testing layer with all required binary utilities pre-provisioned:
```dockerfile
FROM kalilinux/kali-rolling:latest

# Provision all target infrastructure vectors across categories
RUN apt-get update && apt-get install -y \
    # Core Infrastructure & Networking
    kali-tools-top10 \
    python3-pip \
    curl \
    python3-pexpect \
    netcat-openbsd \
    \
    # Reverse Engineering & Binary Analysis
    gdb \
    radare2 \
    binutils \
    ltrace \
    strace \
    file \
    python3-pwntools \
    \
    # Cleanup to keep the image footprint optimized
    && apt-get clean && rm -rf /var/lib/apt/lists/*
```

Compile the image layer layout (this can take a few minutes as it pulls down the Kali repository arrays):
```bash
docker build -t kali-ctf-agent .
```

### 2. Isolate Your Python Virtual Environment
To bypass modern package management locks (`externally-managed-environment` errors), configure an isolated Python workspace wrapper:
```bash
# Initialize the environment wrapper
python3 -m venv ctf_env

# Activate the virtual environment
source ctf_env/bin/activate

# Install the verified framework client packages
pip install groq docker
```

### 3. Expose Your API Key
Export your Groq API authentication key directly into your active console session:
```bash
export GROQ_API_KEY="your_groq_api_key_here"
```
*(Tip: Add this export command to your `~/.bashrc` or `~/.zshrc` file to preserve it permanently across terminal restarts).*

### 4. Create the Master Code Script
Save your interactive python orchestration harness code as `agent_harness.py` inside your `~/agent_harness/` folder.

---

## 🎯 How to Run a Challenge

1. Activate your environment and fire up the engine:
   ```bash
   source ctf_env/bin/activate
   python -u agent_harness.py
   ```
2. Copy and paste your entire challenge overview directly from the CTF page (including target netcat strings, links, and hints).
3. Type **`DONE`** on a new line and press **Enter** to pass the collection sequence over to the agent core.

### Example Console Input:
```text
============================================================
      ENTER YOUR CHALLENGE DETAILS / DESCRIPTION BELOW      
  (Type 'DONE' on a new line and press Enter when finished)  
============================================================

Target service link: nc xebec.cylabacademy.net 31495
Source code download: http://challenge-files.com

Description: The target application parses an interactive lyric loop but contains a hidden code injection logic flaw. Escape the subroutine and extract flag.txt.
DONE
```

