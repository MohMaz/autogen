---
myst:
html_meta:
"description lang=en": |
Quick Start Guide for AgentChat: Migrating from AutoGen 0.2x to 0.4x.
---

# Quick Start

:::{note}
For installation instructions, please refer to the [package information](pkg-info-autogen-agentchat).
:::

AgentChat API, introduced in AutoGen 0.4x, offers a similar level of abstraction as the default Agent classes in AutoGen 0.2x. This guide demonstrates how to migrate basic examples from AutoGen 0.2x to AgentChat in AutoGen 0.4x.

## Migrating from AutoGen 0.2x to 0.4x

The following example illustrates creating a simple agent team with two agents and compares how this is done in AutoGen 0.2x and 0.4x.

1. An `AssistantAgent` (0.2x) / `CodingAssistantAgent` (0.4x) that generates responses using an LLM model.
2. A `UserProxyAgent` (0.2x) / `CodeExecutorAgent` (0.4x) that executes code snippets and returns the output.

The task is to "Create a plot of NVIDIA and TESLA stock returns YTD from 2024-01-01 and save it to 'nvidia_tesla_2024_ytd.png'."

``````{tab-set}

`````{tab-item} AgentChat (v0.4x)
```python
from autogen_agentchat.agents import CodeExecutorAgent, CodingAssistantAgent
from autogen_agentchat.teams.group_chat import RoundRobinGroupChat
from autogen_core.components.code_executor import DockerCommandLineCodeExecutor
from autogen_core.components.models import OpenAIChatCompletionClient

async with DockerCommandLineCodeExecutor(work_dir="coding") as code_executor:
    code_executor_agent = CodeExecutorAgent("code_executor", code_executor=code_executor)
    coding_assistant_agent = CodingAssistantAgent(
        "coding_assistant", model_client=OpenAIChatCompletionClient(model="gpt-4")
    )
    group_chat = RoundRobinGroupChat([coding_assistant_agent, code_executor_agent])
    result = await group_chat.run(
        task="Create a plot of NVIDIA and TESLA stock returns YTD from 2024-01-01 and save it to 'nvidia_tesla_2024_ytd.png'."
    )
    print(result)
```
`````

`````{tab-item} v0.2x
```python
from autogen import AssistantAgent, UserProxyAgent, config_list_from_json

config_list = config_list_from_json(env_or_file="OAI_CONFIG_LIST")
assistant = AssistantAgent("assistant", llm_config={"config_list": config_list})
code_executor_agent = UserProxyAgent(
    "code_executor_agent",
    code_execution_config={"work_dir": "coding", "use_docker": True}
)
code_executor_agent.initiate_chat(
    assistant,
    message="Create a plot of NVIDIA and TESLA stock returns YTD from 2024-01-01 and save it to 'nvidia_tesla_2024_ytd.png'."
)
```
`````

``````

AgentChat in v0.4x provides similar abstractions to the default agents in v0.2x:

- `CodingAssistantAgent` (v0.4x) ~ `AssistantAgent` (v0.2x)
- `CodeExecutorAgent` (v0.4x) ~ `UserProxyAgent` with code execution (v0.2x)

Key differences:

1. In v0.4x, agent interactions are managed by `Teams` (e.g., `RoundRobinGroupChat`), replacing direct chat initiation.
2. v0.4x uses async/await syntax for improved performance and scalability.
3. Configuration in v0.4x is more modular, with separate components for code execution and LLM clients.