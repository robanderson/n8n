# Plan: Enabling Multiple Independent Chat Model Instances in n8n AI Agent

**Objective:** Modify the n8n AI Agent node to support multiple, independently configurable instances of the Ollama and OpenAI chat models, and add a new "LM Studio" chat model based on Ollama.

**Problem:** Currently, adding multiple instances of the same chat model node type (e.g., two "Ollama Chat Model" nodes) results in linked configurations – changing one affects the others.

**Solution:** Create new, distinct n8n node types for each required model instance. Each new type will have a unique internal name and display name, allowing for independent configuration.

**Steps:**

1.  **Create New Node Type Directories:**
    *   Navigate to `packages/@n8n/nodes-langchain/nodes/llms/`.
    *   Copy the entire `LMChatOllama/` directory and rename the copy to `LMChatLmStudio/`.
    *   Copy `LMChatLmStudio/` and rename the copy to `LMChatLmStudio1/`.
    *   Copy `LMChatLmStudio/` and rename the copy to `LMChatLmStudio2/`.
    *   Copy `LMChatOllama/` and rename the copy to `LMChatOllama1/`.
    *   Copy `LMChatOllama/` and rename the copy to `LMChatOllama2/`.
    *   Copy the entire `LMChatOpenAi/` directory and rename the copy to `LMChatOpenAi1/`.
    *   Copy `LMChatOpenAi/` and rename the copy to `LMChatOpenAi2/`.

2.  **Modify New Node Files:** For *each* newly created directory (`LMChatLmStudio`, `LMChatLmStudio1`, `LMChatLmStudio2`, `LMChatOllama1`, `LMChatOllama2`, `LMChatOpenAi1`, `LMChatOpenAi2`):
    *   **Rename File:** Inside the directory, rename the `.node.ts` file to match the directory name (e.g., `LMChatLmStudio.node.ts`, `LMChatOllama1.node.ts`, `LMChatOpenAi1.node.ts`).
    *   **Update Class Name:** Open the renamed `.node.ts` file. Change the exported class name to match the new file name (e.g., `export class LmChatLmStudio implements INodeType { ... }`, `export class LmChatOllama1 implements INodeType { ... }`).
    *   **Update Node Name:** Find the `description` object within the class definition. Change the `name` property to a unique identifier, conventionally matching the class name but lower-cased (e.g., `name: 'lmChatLmStudio'`, `name: 'lmChatOllama1'`, `name: 'lmChatOpenAi1'`).
    *   **Update Display Name:** In the same `description` object, change the `displayName` property to the desired user-facing name:
        *   For `LMChatLmStudio.node.ts`: `'LM Studio Chat Model'`
        *   For `LMChatLmStudio1.node.ts`: `'LM Studio Chat Model 1'`
        *   For `LMChatLmStudio2.node.ts`: `'LM Studio Chat Model 2'`
        *   For `LMChatOllama1.node.ts`: `'Ollama Chat Model 1'`
        *   For `LMChatOllama2.node.ts`: `'Ollama Chat Model 2'`
        *   For `LMChatOpenAi1.node.ts`: `'OpenAI Chat Model 1'`
        *   For `LMChatOpenAi2.node.ts`: `'OpenAI Chat Model 2'`
    *   **Set LM Studio Default URL:** In `LMChatLmStudio.node.ts`, `LMChatLmStudio1.node.ts`, and `LMChatLmStudio2.node.ts`, locate the node property definition for the base URL (it's likely named `baseUrl` within the `properties` array). Add or modify the `default` value for this property to `'http://127.0.0.1:1234'`.

3.  **Update AI Agent Node (`Agent.node.ts`):**
    *   Open `packages/@n8n/nodes-langchain/nodes/agents/Agent/Agent.node.ts`.
    *   Locate the `getInputs` function definition near the top of the file.
    *   Inside this function, find the `filter.nodes` arrays associated with the `type: 'ai_languageModel'` input configuration for the relevant agent types (specifically `conversationalAgent`, `toolsAgent`, and potentially others if you use them).
    *   Add the fully qualified names (package name + node name) of the *new* node types to these arrays. Ensure you add them to all relevant agent filters:
        *   `'@n8n/n8n-nodes-langchain.lmChatLmStudio'`
        *   `'@n8n/n8n-nodes-langchain.lmChatLmStudio1'`
        *   `'@n8n/n8n-nodes-langchain.lmChatLmStudio2'`
        *   `'@n8n/n8n-nodes-langchain.lmChatOllama1'`
        *   `'@n8n/n8n-nodes-langchain.lmChatOllama2'`
        *   `'@n8n/n8n-nodes-langchain.lmChatOpenAi1'`
        *   `'@n8n/n8n-nodes-langchain.lmChatOpenAi2'`

4.  **Build Project:**
    *   Navigate to the root directory of the n8n project (`/Users/robanderson/Documents/Development/n8n/n8n`).
    *   Run the project's build command. This is typically `pnpm build` or `npm run build`. This command will compile the TypeScript code, and the build system should automatically detect and register the new node types.

5.  **Test:**
    *   Start n8n (e.g., `npm run start` or however you usually launch it).
    *   Create a new workflow.
    *   Add an "AI Agent" node.
    *   Attempt to connect a chat model to the "Chat Model" input (or "Model" input depending on the agent type). Verify that all the new models ("LM Studio Chat Model", "LM Studio Chat Model 1", "Ollama Chat Model 1", "OpenAI Chat Model 2", etc.) are listed as available options alongside the originals.
    *   Add multiple instances of the *same* new node type (e.g., two "LM Studio Chat Model 1" nodes) and different new node types onto the canvas.
    *   Configure each instance with distinct settings (e.g., different API keys for OpenAI, different model names within Ollama/LM Studio).
    *   Confirm that changing the settings in one node instance does *not* affect the settings of any other instance.
