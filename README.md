# Agent Data Injection (ADI) Attack Benchmark

This repository contains the artifacts for the paper *"Agent Data Injection Attacks are Realistic Threats to AI Agents"*. It provides two benchmarks for evaluating Agent Data Injection (ADI) attacks on LLMs and LLM agents.

## What is Agent Data Injection (ADI)?

Agent Data Injection (ADI) is a novel attack that exploits how LLM agents process structured data. Unlike traditional prompt injection attacks that embed explicit malicious instructions, ADI attacks inject malformed data that causes parsing confusion, leading agents to incorrectly propagate injected values.

**Key characteristics:**
- No explicit malicious instructions in the payload
- Exploits structural syntax to override fields through parsing confusion
- Tests whether agents properly isolate injected data from legitimate environment data

**Example:** An ADI attack might inject a malformed calendar event that causes the agent to extract an attacker-controlled email address instead of the legitimate one.

## Repository Contents

This repository contains two evaluations corresponding to the paper:

- **LLM evaluation** (`llm-benchmark/`): Evaluates standalone LLM susceptibility to ADI attacks on structured tool outputs (calendar, email, GitHub issues, Slack, cloud drive, web search, etc.) across multiple data formats (JSON, HTML, Markdown). See `llm-benchmark/README.md` for details.
- **End-to-end agent evaluation** (`agentdojo/` + `agents/`): Extends AgentDojo with ADI attacks and ports nine existing prompt-injection defenses (Llama Prompt Guard, LlamaFirewall, Progent, IsolateGPT, CaMeL variants, etc.) for systematic comparison.

Real-world attack demonstrations (Section 4 in the paper) are not provided as executable scripts to prevent misuse.

## Installation

Both benchmarks use [uv](https://github.com/astral-sh/uv) for dependency management.

```bash
# Clone the repository
git clone https://github.com/compsec-snu/adi.git
cd adi

# Install dependencies for the end-to-end agent benchmark
cd agentdojo && uv sync && cd ..

# Install dependencies for the LLM benchmark
cd llm-benchmark && uv sync && cd ..
```

## Environment Variables

Set the appropriate API keys for the models you want to use:

```bash
export OPENAI_API_KEY="your-openai-key"
export ANTHROPIC_API_KEY="your-anthropic-key"
export GOOGLE_API_KEY="your-google-key"
```

## Running the LLM Benchmark

See `llm-benchmark/README.md` for the full design and configuration reference. Quick start:

```bash
cd llm-benchmark
uv run dib --help
```

The benchmark is configured via YAML files in `configs/` (`benchmark_quick.yaml`, `benchmark_full.yaml`, `benchmark_syntactic.yaml`).

## Running the End-to-End Agent Benchmark

### Basic Usage

Run the ADI attack benchmark with:

```bash
uv run python -m agentdojo.scripts.benchmark \
    --agent <agent_name> \
    --model <model_name> \
    --attack data_only_syntactic \
    -s <suite_name>
```

### Examples

```bash
# Run ADI attack on workspace suite with baseline agent
uv run python -m agentdojo.scripts.benchmark \
    --agent baseline \
    --model gpt-4o-mini \
    --attack data_only_syntactic \
    -s workspace

# Run on multiple suites
uv run python -m agentdojo.scripts.benchmark \
    --agent baseline \
    --model gpt-4o-mini \
    --attack data_only_syntactic \
    -s workspace -s banking -s slack -s travel

# Run specific user tasks
uv run python -m agentdojo.scripts.benchmark \
    --agent baseline \
    --model gpt-4o-mini \
    --attack data_only_syntactic \
    -s workspace \
    -ut user_task_0 -ut user_task_1

# Force re-run existing results
uv run python -m agentdojo.scripts.benchmark \
    --agent baseline \
    --model gpt-4o-mini \
    --attack data_only_syntactic \
    -s workspace \
    --force-rerun
```

### Command-Line Options

| Option | Description |
|--------|-------------|
| `--agent` | Agent to evaluate (see available agents below) |
| `--model` | LLM model identifier |
| `--attack` | Attack type (`data_only_syntactic` for ADI) |
| `-s, --suite` | Task suite(s) to run (can specify multiple) |
| `-ut, --user-task` | Specific user task(s) to run |
| `-it, --injection-task` | Specific injection task(s) to run |
| `--logdir` | Directory for output logs (default: `./runs`) |
| `-f, --force-rerun` | Force re-execution of existing results |
| `--help` | Show all available options |

## Available Task Suites

| Suite | Description |
|-------|-------------|
| `workspace` | Email, calendar, and cloud drive tasks |
| `banking` | Banking and financial transactions |
| `slack` | Slack messaging and channel operations |
| `travel` | Travel booking and reservations |

## Available Agents

### LangGraph Agents
- `baseline` - Standard ReAct-style agent
- `baseline_random_suffix` - Baseline with random suffix defense
- `llama_prompt_guard` - Agent with Llama Prompt Guard
- `llama_firewall` - Agent with Llama Firewall
- `isolate_gpt` - Agent with isolation-based defense

### Pipeline Agents
- `camel` - CAMEL agent implementation
- `camel_secpol` - CAMEL with security policy
- `camel_secpol_strict` - CAMEL with strict security policy
- `progent` - Progent agent implementation

## Supported Models

The benchmark supports various LLM providers:

- **OpenAI**: `gpt-4o`, `gpt-4o-mini`, `gpt-4-turbo`, etc.
- **Anthropic**: `claude-3-5-sonnet`, `claude-3-opus`, etc.
- **Google**: `gemini-1.5-pro`, `gemini-1.5-flash`, etc.

## Output

Results are saved to the `--logdir` directory (default: `./runs`) in JSON format, containing:
- Task execution traces
- Utility scores (task completion)
- Security scores (attack prevention)

## Project Structure

```
adi/
├── README.md              # This file
├── LICENSE
├── llm-benchmark/         # Standalone LLM evaluation
│   ├── README.md          # Detailed design and usage
│   ├── src/data_injection_bench/
│   ├── configs/           # Benchmark + attack + category configs
│   ├── data/              # Seed samples + collection scripts
│   ├── prompts/           # LLM prompts
│   ├── scripts/           # Helper scripts
│   └── pyproject.toml
├── agentdojo/             # AgentDojo benchmark framework (forked)
│   ├── src/agentdojo/
│   │   ├── attacks/       # Attack implementations (including ADI)
│   │   ├── agents/        # Agent loading system
│   │   ├── default_suites/# Task suite definitions
│   │   └── scripts/       # Benchmark CLI
│   └── pyproject.toml
└── agents/                # Agent / defense implementations
    ├── baseline/
    ├── baseline_random_suffix/
    ├── isolate_gpt/
    ├── llama_firewall/
    ├── llama_prompt_guard/
    ├── camel/
    ├── camel_secpol/
    ├── camel_secpol_strict/
    └── progent/
```

## Citation

If you use this benchmark in your research, please cite our paper:

```bibtex
@misc{choi2026adi,
  title        = {Agent Data Injection Attacks are Realistic Threats to AI Agents},
  author       = {Woohyuk Choi and Juhee Kim and Taehyun Kang and Jihyeon Jeong and Luyi Xing and Byoungyoung Lee},
  year         = {2026},
  eprint       = {2607.05120},
  archivePrefix= {arXiv},
  primaryClass = {cs.CR}
}
```

## License

This project is released under the MIT License. See [LICENSE](LICENSE) for details.
