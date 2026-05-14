# agentic-workbench

_Prototype. Orchestrate. Accelerate._

Reusable code recipes and integration patterns leveraging Copilot, agents, and AI services to streamline developer workflows.

## Plugins

The [`plugins/`](plugins/) directory contains self-contained VS Code Copilot plugins. Each plugin follows a consistent layout:

- `plugin.json` — manifest describing the plugin (name, description, version, author) and pointing to its `skills/` and `agents/` directories.
- `skills/` — domain-specific skill instructions Copilot can load on demand.
- `agents/` — custom agent definitions (`*.agent.md`) that orchestrate skills and tools for a focused task.

### Available plugins

| Plugin | Description |
| --- | --- |
| [llm-cost-estimator](plugins/llm-cost-estimator/) | Estimate input/output token cost of prompts across OpenAI models using `tiktoken` and live pricing from developers.openai.com. Ships the `openai-models-cost-estimator` skill and the `llm-prompt-cost` agent. |
| [cost-estimators](plugins/cost-estimators/) | Cost estimation toolkit for AI workloads. Bundles the `openai-models-cost-estimator` and `fda-cost-estimation-skill` skills with the `llm-prompt-cost` and `fda-cost-agent` agents to estimate OpenAI prompt token costs and Microsoft Fabric Data Agent CU consumption (with SKU recommendations). |

## Install plugins in VS Code

These plugins follow the [VS Code agent plugin](https://code.visualstudio.com/docs/copilot/customization/agent-plugins) format (currently in **Preview**). Make sure `chat.plugins.enabled` is `true` in your VS Code settings before installing.

> The plugins live in subdirectories of this repo (`plugins/<plugin-name>/`), so each one is registered individually rather than via a single root-level `plugin.json`.

### Installation options

| #   | Method                                                                                  | Best for                                                      |
| --- | --------------------------------------------------------------------------------------- | ------------------------------------------------------------- |
| 1   | [Clone & register locally](#option-1--clone-and-register-as-local-plugins-recommended)  | Full control, offline use, active development *(recommended)* |
| 2   | [Plugin marketplace](#option-2--add-this-repo-as-a-plugin-marketplace)                  | Discovering and managing plugins via the Extensions view      |
| 3   | [Install from source](#option-3--install-a-single-plugin-from-source)                   | Quick one-off install of a single plugin                      |

---

### Option 1 — Clone and register as local plugins (recommended)

1. Clone this repository:

   ```pwsh
   git clone https://github.com/raaravap_microsoft/agentic-workbench.git
   ```

2. In VS Code, open **Settings (JSON)** and add each plugin folder you want to enable to [`chat.pluginLocations`](https://code.visualstudio.com/docs/copilot/customization/agent-plugins#_use-local-plugins):

   ```jsonc
   // settings.json
   "chat.pluginLocations": {
     "C:/path/to/agentic-workbench/plugins/llm-cost-estimator": true,
     "C:/path/to/agentic-workbench/plugins/cost-estimators": true
   }
   ```

   Set the value to `false` to keep a plugin registered but disabled.

3. Reload the window. The plugin's skills and agents now appear in the Chat view and the **Agent Plugins – Installed** section of the Extensions view (`@agentPlugins` filter).

### Option 2 — Add this repo as a plugin marketplace

If you prefer to discover plugins through the Extensions view, add this repository as a marketplace via the [`chat.plugins.marketplaces`](https://code.visualstudio.com/docs/copilot/customization/agent-plugins#_configure-plugin-marketplaces) setting:

```jsonc
// settings.json
"chat.plugins.marketplaces": [
  "raaravap_microsoft/agentic-workbench"
  // or the full git remote:
  // "https://github.com/raaravap_microsoft/agentic-workbench.git"
]
```

Then open the Extensions view, search for `@agentPlugins`, and install the plugins you want. The first time you add a new marketplace, VS Code shows a trust prompt — review the source before confirming.

### Option 3 — Install a single plugin from source

You can also install directly from a Git URL using **Chat: Install Plugin From Source** (Command Palette) and pointing it at this repository. See [Install a plugin from source](https://code.visualstudio.com/docs/copilot/customization/agent-plugins#_install-a-plugin-from-source) for details.

---

For more on managing, updating, and disabling plugins, see the official VS Code docs: <https://code.visualstudio.com/docs/copilot/customization/agent-plugins#_configure-plugin-marketplaces>.

---

## Troubleshooting

### A new or updated plugin doesn't appear after the marketplace was updated

VS Code's **Agent Plugins** marketplace clones each repository to a local cache the first time it's added, but the **refresh button does not pull updates** from the remote. As a result, newly added plugins (or version bumps) in a marketplace you've already added may not show up. The fix is to delete the cached clone and let VS Code re-clone it.

The cache lives at:

| OS                  | Path                                                                            |
| ------------------- | ------------------------------------------------------------------------------- |
| **Windows**         | `%USERPROFILE%\.vscode\agent-plugins\github.com\<owner>\<repo>`                 |
| **macOS / Linux**   | `~/.vscode/agent-plugins/github.com/<owner>/<repo>`                             |

For this repo, replace `<owner>/<repo>` with `raaravap_microsoft/agentic-workbench`.

#### Windows (PowerShell)

```pwsh
Remove-Item "$env:USERPROFILE\.vscode\agent-plugins\github.com\raaravap_microsoft\agentic-workbench" -Recurse -Force
```

#### macOS / Linux (bash/zsh)

```bash
rm -rf ~/.vscode/agent-plugins/github.com/raaravap_microsoft/agentic-workbench
```

Then in VS Code:

1. Run **Developer: Reload Window** from the Command Palette.
2. Open the Extensions view, filter with `@agentPlugins`, and the new/updated plugin should appear.

> **Tip for active development:** if you're authoring or frequently editing plugins in this repo, skip the marketplace cache entirely. Clone the repo locally and register the plugin folders directly via [`chat.pluginLocations`](https://code.visualstudio.com/docs/copilot/customization/agent-plugins#_use-local-plugins) (see [Option 1](#option-1--clone-and-register-as-local-plugins-recommended)). Local plugins are read straight from disk on every reload — no clone, no cache, no refresh dance.

