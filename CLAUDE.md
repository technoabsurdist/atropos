# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Installation and Setup
```bash
# Install for development
pip install -e ".[dev]"

# Install pre-commit hooks (required for contributors)
pre-commit install

# Run pre-commit checks manually
pre-commit run --all-files
```

### Testing
```bash
# Run all tests
pytest

# Run tests with provider API keys (requires keys)
pytest --runproviders

# Run specific test file
pytest atroposlib/tests/test_advantages.py
```

### Code Quality
```bash
# Format code with black
black .

# Sort imports
isort --profile black .

# Lint with flake8
flake8 --max-line-length=120 --extend-ignore=E203,W503
```

### Running Atropos

#### Start the API Server
```bash
# Start the central rollout API server
run-api
```

#### Run an Environment
```bash
# Run GSM8K environment (in separate terminal)
python environments/gsm8k_server.py serve --openai.model_name Qwen/Qwen2.5-1.5B-Instruct --slurm false

# Run with config file
python environments/gsm8k_server.py serve --config environments/configs/example.yaml

# Override config with CLI args
python environments/gsm8k_server.py serve --config environments/configs/example.yaml --env.group_size 8
```

#### Generate Training Data
```bash
# Generate SFT data
atropos-sft-gen path/to/output.jsonl --tokenizer Qwen/Qwen2.5-1.5B-Instruct

# Generate DPO data
atropos-dpo-gen path/to/output.jsonl --tokenizer Qwen/Qwen2.5-1.5B-Instruct
```

#### Debugging Tools
```bash
# Process environment locally (no API server needed)
python environments/gsm8k_server.py process --env.data_path_to_save_groups gsm8k_rollouts.jsonl

# View rollouts in Gradio UI
view-run

# View multimodal rollouts
view-run-multimodal
```

## Architecture Overview

### Core Components

**Atropos** is a microservice framework for asynchronous RL with LLMs that decouples environments from trainers.

#### 1. API Server (`atroposlib/api/server.py`)
- Central FastAPI server that coordinates rollout collection and distribution
- Manages heterogeneous batching from multiple environments
- Provides REST endpoints for trainers to pull batches and environments to submit rollouts
- Run with `run-api` command

#### 2. Base Environment (`atroposlib/envs/base.py`)
- Abstract base class `BaseEnv` that all environments inherit from
- Handles OpenAI API communication, scoring, batching, and WandB logging
- Uses `BaseEnvConfig` for configuration management
- Supports both synchronous and asynchronous operations

#### 3. Environment Server Handling (`atroposlib/envs/server_handling/`)
- **ServerManager**: Discovers and manages multiple inference servers (local vLLM/SGLang or cloud APIs)
- **OpenAI Server**: Handles communication with OpenAI-compatible APIs
- **Server Baseline**: Configuration for server discovery and load balancing

#### 4. Environment Implementations (`environments/`)
- Ready-to-use environments like GSM8K, tool calling, RLAIF, multimodal tasks
- Community contributions in `environments/community/`
- Each environment defines its own scoring logic and data processing

#### 5. Example Trainer (`example_trainer/grpo.py`)
- Reference implementation showing how to integrate with Atropos API
- Demonstrates GRPO (Generalized Proximal Policy Optimization) training loop
- Pulls batches from API server and performs model updates

### Data Flow

1. **Environment** generates prompts and sends them to inference servers
2. **Inference servers** (vLLM/SGLang/OpenAI) generate responses
3. **Environment** scores responses and groups them for training
4. **Environment** sends scored groups to the **API server**
5. **Trainer** pulls batches from the **API server**
6. **Trainer** performs RL updates and pushes new model checkpoints to inference servers

### Key Design Patterns

#### Asynchronous and Decoupled
- Environments run independently as microservices
- Multiple environments can contribute to the same training run
- Trainers pull data when ready (no blocking)

#### Inference Agnostic
- Supports any OpenAI-compatible API (vLLM, SGLang, cloud providers)
- Easy switching between local and cloud inference
- Configurable server discovery and load balancing

#### Heterogeneous Batching
- API server combines rollouts from different environments
- Enables multi-task and multi-modal training
- Weighted sampling based on environment priorities

## Configuration System

Atropos uses Pydantic models with CLI integration via `pydantic-cli`. All config classes support:
- YAML file configuration
- CLI argument overrides
- Namespace-based organization (`--env.param`, `--openai.param`)

Key config classes:
- `BaseEnvConfig`: Core environment settings (batch size, tokenizer, WandB)
- `APIServerConfig`: OpenAI API server configuration
- `ServerManagerConfig`: Server discovery and SLURM settings

See `CONFIG.md` for complete configuration options.

## Testing Infrastructure

- Uses `pytest` with async support
- Provider tests require `--runproviders` flag and API keys
- Mock data available for testing without real API calls
- Test configuration in `pytest.ini`

## Development Workflow

1. **Environment Development**: Inherit from `BaseEnv`, implement scoring logic
2. **Local Testing**: Use `process` command to test environments locally
3. **Integration**: Connect to API server and run with trainers
4. **Scaling**: Deploy multiple environment instances across nodes

## Contribution Guidelines

- New environments go in `environments/community/`
- Follow import conventions treating environment directory as package root
- All contributions must pass pre-commit hooks (black, isort, flake8)
- Use PR templates for environment vs non-environment changes
- Ensure legal compliance and appropriate content guidelines