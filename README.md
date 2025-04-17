# T-Agent v0.0.1: Building a Basic Local AI Agent in Python: A non-interactive, text-based course to get you started writing agents

## Introduction

This course provides a focused, no-frills, accelerated pathway to writing a simple but functional locally hosted AI agent in Python. It walks a novice agent enthusiast through:

* architectural decisions, to provide the learners with systems-level understanding of the agent 
* functional increments, building up the solution as we go along 
* code implementation, with explanation 
* conceptual diagrams related to code
* provides exercises for understanding
* references for further study

The *initial* purpose of the agent is simply to carry out analysis of a macosx performance and discuss it with the user in a way which is understandable, providing advice on further action.
The intended end state of the agent is to provide an a.i-enhanced "systems administrator" for the local computer (macosx, linux, bsd, unix, and god-forbid -  ms-windows).

**Note: This course is a short as possible to bootstrap the learner as quickly as possible. It's only meant to provide a working starting point which you must then flesh out with solid theoretical foundation and moving on to more complex applications of agentic a.i.**

---

## Part 1: Understanding the Architecture

### 1.1 Agent Architecture Overview

**Goal**: Build an AI agent that communicates via terminal, uses OpenAI's API as its LLM engine, retains local memory, and reports on system performance.

**Architecture Components**:

- **CLI Interface**: Starts and interacts through macOS Terminal
- **OpenAI LLM Client**: Communicates with the OpenAI API
- **System Metrics Monitor**: Uses `psutil` to gather system performance
- **Disk Memory Context**: Saves conversations locally (e.g., JSON format)

```
+-----------+      +--------------+      +-----------------+
| macOS CLI | <--> | Python Agent | <--> | OpenAI LLM API  |
+-----------+      +--------------+      +-----------------+
                      |  ^
                      v  |
                 +-----------+
                 | Context DB|
                 +-----------+
                      |
                      v
                +------------------+
                | System Monitor   |
                +------------------+
```

**Why this design?**
- Modularity allows independent testing
- Clear separation of concerns (I/O, logic, LLM, memory, metrics)

### Reference
- Blog: [OpenAI Python Quickstart](https://platform.openai.com/docs/quickstart)
- Library: [`psutil`](https://github.com/giampaolo/psutil)

---

## Part 2: Project Setup

### 2.1 Initial Project Files
Create the following files:
- `agent.py`
- `memory.json`
- `config.py`

### 2.2 Python Requirements
Install dependencies:
```bash
pip install openai psutil
```

### 2.3 API Key Configuration
```python
# config.py
OPENAI_API_KEY = 'your-api-key-here'
```

**Exercise**:
- Why is it important to separate config from logic?

---

## Part 3: Terminal Agent Shell (macOS CLI Component)

### Diagram: Emphasized Component - CLI Interface
```
+-----------+      +--------------+      +-----------------+
| macOS CLI | <--> | Python Agent |      |                 |
+-----------+      +--------------+      +-----------------+
                      |  ^
                      v  |
                 +-----------+
                 |           |
                 +-----------+
                      |
                      v
                +------------------+
                |                  |
                +------------------+
```

### 3.1 agent.py (CLI Shell)
```python
# agent.py
import openai
import psutil
import json
from config import OPENAI_API_KEY

openai.api_key = OPENAI_API_KEY

def load_memory():
    try:
        with open('memory.json', 'r') as f:
            return json.load(f)
    except:
        return []

def save_memory(memory):
    with open('memory.json', 'w') as f:
        json.dump(memory, f, indent=2)

def get_user_input():
    return input("You: ")
```

### Reference
- Blog: [Using OpenAI’s Completion API](https://platform.openai.com/docs/guides/gpt)

---

## Part 4: Communicating with OpenAI (LLM Client Component)

### Diagram: Emphasized Component - OpenAI LLM API
```
+-----------+      +--------------+      +-----------------+
|           |      | Python Agent | <--> | OpenAI LLM API  |
+-----------+      +--------------+      +-----------------+
                      |  ^
                      v  |
                 +-----------+
                 |           |
                 +-----------+
                      |
                      v
                +------------------+
                |                  |
                +------------------+
```

### 4.1 Add LLM Request Function
```python
def query_llm(prompt):
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": prompt}]
    )
    return response['choices'][0]['message']['content']
```

### 4.2 Add Message Loop
```python
def main():
    memory = load_memory()
    while True:
        user_input = get_user_input()
        if user_input.strip().lower() == 'status':
            metrics = get_system_metrics()
            print(summarize_metrics(metrics))
            continue
        memory.append({"role": "user", "content": user_input})
        response = query_llm(user_input)
        print("Agent:", response)
        memory.append({"role": "assistant", "content": response})
        save_memory(memory)

if __name__ == '__main__':
    main()
```

### Exercise
1. How is memory persisted?
2. What happens if the JSON file is corrupted?
3. How would you allow the user to exit cleanly?

---

## Part 5: Gathering System Performance (System Monitor Component)

### Diagram: Emphasized Component - System Monitor
```
+-----------+      +--------------+      +-----------------+
|           |      | Python Agent |      |                 |
+-----------+      +--------------+      +-----------------+
                      |  ^
                      v  |
                 +-----------+
                 |           |
                 +-----------+
                      |
                      v
                +------------------+
                | System Monitor   |
                +------------------+
```

### 5.1 System Metrics Function
```python
def get_system_metrics():
    return {
        "cpu_percent": psutil.cpu_percent(),
        "virtual_memory": psutil.virtual_memory()._asdict(),
        "disk_usage": psutil.disk_usage('/')._asdict(),
        "top_processes": sorted([
            (p.pid, p.name(), p.cpu_percent(), p.memory_info().rss)
            for p in psutil.process_iter(['name'])
        ], key=lambda x: x[2], reverse=True)[:5]
    }
```

### Exercise
- How would you translate system metrics into plain English?
- How can you avoid printing large memory numbers in bytes?

---

## Part 6: Human-Readable Summaries (Summary Layer)

### Diagram: Emphasized Component - Human Summary Function
```
+-----------+      +--------------+      +-----------------+
|           |      | Python Agent | <--> | OpenAI LLM API  |
+-----------+      +--------------+      +-----------------+
                      |  ^
                      v  |
                 +-----------+
                 | Context DB|
                 +-----------+
                      |
                      v
                +------------------+
                | System Monitor   |
                +------------------+
                      |
                      v
                +------------------+
                |   Summary Logic  |
                +------------------+
```

```python
def summarize_metrics(metrics):
    return (
        f"CPU Usage: {metrics['cpu_percent']}%\n"
        f"Memory Used: {metrics['virtual_memory']['percent']}%\n"
        f"Disk Used: {metrics['disk_usage']['percent']}%\n"
        f"Top processes:\n" +
        '\n'.join([f"- {name} (PID {pid}) using {cpu}% CPU" for pid, name, cpu, _ in metrics['top_processes']])
    )
```

---

## Summary & Final Questions

1. What core architectural decisions were made and why?
2. How does local memory persistence benefit the agent?
3. What would be your next improvement: web GUI, multithreading, logging?

---

## References
1. OpenAI Quickstart - https://platform.openai.com/docs/quickstart
2. psutil Python Library - https://github.com/giampaolo/psutil
3. JSON Persistence Guide - https://realpython.com/python-json/

---

## Next Steps

Here are some possible enhancements you can explore:

1. Add support for natural language questions like "How much memory am I using right now?"
2. Implement logging to a file for all agent interactions.
3. Improve the memory system with embeddings for semantic context.
4. Add a configuration menu or CLI flags for custom commands.
5. Add error handling with retries for API calls.
6. Encrypt stored memory for data privacy.
7. Implement a GUI using Tkinter or a web interface.
8. Add multi-threading support for performance monitoring in the background.
9. Integrate additional APIs (e.g., weather, calendar) to extend assistant functionality.

* Raise the bar:

10. Rewrite this agent with reduced reliance on existing frameworks and APIs
11. Use your own custom written LLM, fin tuned on data from your local machine
12. Rewrite the agent in another language: Golang, Rust, C, C++
13. Empower the agent to recommend technical performance tuning configuration changes to the local machine
14. Add a security feature for baselining performance for detection of security anomalies
15. Make the agent "multi-agent", splitting it into a team of smaller, collaborating agents using A2A
16. Integrate MCP into the collective agent to make the agent extensible via MCP
17. Tune the codebase to be as minimal as possible
18. Develop an MLOps solution to ensure full CI/CD management of the codebase and the fine-tuning data for the agent.
19. Include a feature to swap between LLMaaS (OpenAi, DeepSeek, Mistral etc ...)
20. Make the LLM fully local

---

## Appendix: Complete Code for Reference

* Tying it all together, your code should look like this (more or less):


```python
# config.py
OPENAI_API_KEY = 'your-api-key-here'
```

```python
# agent.py
import openai
import psutil
import json
from config import OPENAI_API_KEY

openai.api_key = OPENAI_API_KEY

def load_memory():
    try:
        with open('memory.json', 'r') as f:
            return json.load(f)
    except:
        return []

def save_memory(memory):
    with open('memory.json', 'w') as f:
        json.dump(memory, f, indent=2)

def get_user_input():
    return input("You: ")

def query_llm(prompt):
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[{"role": "user", "content": prompt}]
    )
    return response['choices'][0]['message']['content']

def get_system_metrics():
    return {
        "cpu_percent": psutil.cpu_percent(),
        "virtual_memory": psutil.virtual_memory()._asdict(),
        "disk_usage": psutil.disk_usage('/')._asdict(),
        "top_processes": sorted([
            (p.pid, p.name(), p.cpu_percent(), p.memory_info().rss)
            for p in psutil.process_iter(['name'])
        ], key=lambda x: x[2], reverse=True)[:5]
    }

def summarize_metrics(metrics):
    return (
        f"CPU Usage: {metrics['cpu_percent']}%\n"
        f"Memory Used: {metrics['virtual_memory']['percent']}%\n"
        f"Disk Used: {metrics['disk_usage']['percent']}%\n"
        f"Top processes:\n" +
        '\n'.join([f"- {name} (PID {pid}) using {cpu}% CPU" for pid, name, cpu, _ in metrics['top_processes']])
    )

def main():
    memory = load_memory()
    while True:
        user_input = get_user_input()
        if user_input.strip().lower() == 'status':
            metrics = get_system_metrics()
            print(summarize_metrics(metrics))
            continue
        memory.append({"role": "user", "content": user_input})
        response = query_llm(user_input)
        print("Agent:", response)
        memory.append({"role": "assistant", "content": response})
        save_memory(memory)

if __name__ == '__main__':
    main()
```

