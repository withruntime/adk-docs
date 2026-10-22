---
catalog_title: Runtime
catalog_description: Run model-written code and shell commands in Firecracker microVM sandboxes
catalog_icon: /integrations/assets/runtime.png
catalog_tags: ["code"]
---

# Runtime code executor and toolset for ADK

<div class="language-support-tag">
  <span class="lst-supported">Supported in ADK</span><span class="lst-python">Python</span>
</div>

The [google-adk-withruntime](https://pypi.org/project/google-adk-withruntime/)
package connects your ADK agent to [Runtime](https://withruntime.com)
sandboxes. Each sandbox is a Firecracker microVM with its own Linux kernel.
It adds a code executor, for agents whose model writes Python blocks, and a
toolset, for agents that call shell and file tools.

## Use cases

- **Code execution without Docker**: Use `RuntimeCodeExecutor` where you would
  use `ContainerCodeExecutor`, with no Docker daemon or cluster to run.

- **Coding agents**: Let the model run commands, install packages, and read
  and write files in a full Linux machine with passwordless `sudo`.

- **Data analysis**: Files written by one code block are there for the next,
  so an agent can load data once and build on it.

## Prerequisites

- A [Runtime](https://withruntime.com) account. New accounts get 100 free
  hours of a 1 GB machine in their first 30 days.
- A Runtime API key, or a machine signed in with `npx withruntime login`

## Installation

```bash
pip install google-adk-withruntime
```

## Use with agent

```python
from google.adk.agents import Agent
from google_adk_withruntime import RuntimeCodeExecutor, RuntimeToolset

# The model writes Python blocks; ADK runs them in the sandbox.
analyst = Agent(
    model="gemini-flash-latest",
    name="analyst",
    instruction="Answer data questions by writing and running Python.",
    code_executor=RuntimeCodeExecutor(),  # Reads RUNTIME_API_KEY
)

# The model calls shell and file tools in the sandbox.
coder = Agent(
    model="gemini-flash-latest",
    name="coder",
    instruction="Solve tasks by running commands in the sandbox.",
    tools=[RuntimeToolset()],
)
```

## Available tools

The `RuntimeToolset` class provides:

Tool | Description
---- | -----------
`runtime_exec` | Run a shell command and return its exit code, stdout and stderr
`runtime_read_file` | Read a text file
`runtime_write_file` | Create or replace a text file
`runtime_list_files` | List a directory with each entry's type and size

## Configuration

- A code block that runs past `RuntimeCodeExecutor(timeout_seconds=300)` is
  killed and reports exit code 124.
- Pass `create={"vcpu": 2, "image": "my-image:v1"}` to either class to set the
  sandbox's size, image and network rules.
- Pass `RuntimeCodeExecutor(sandbox_id=...)` or `RuntimeToolset(sandbox)` to
  use a sandbox you own, which is never stopped for you.
- A sandbox the package starts has a 30-minute lease that ends in a stop, and
  it stops when the executor or the runner's toolsets are closed.

## Additional resources

- [google-adk-withruntime on PyPI](https://pypi.org/project/google-adk-withruntime/)
- [google-adk-withruntime on GitHub](https://github.com/withruntime/runtime/tree/main/integrations/google-adk)
- [Runtime framework guide](https://withruntime.com/docs/frameworks#google-adk)
- [Runtime documentation](https://withruntime.com/docs)
