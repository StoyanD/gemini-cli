# Plan: Evolving Gemini CLI Extensions with Enhanced Capabilities

This document outlines a plan to enhance the Gemini CLI's extension system to be
more powerful and flexible.

## 1. Introduction

The goal is to enhance the existing extension system to allow extensions to
provide not just tools and context, but also a more structured way for the model
to discover and use them. This will enable the creation of more powerful,
domain-specific extensions.

The key principles of this enhanced system are:

- **Structured Extension Definition**: Extensions will be defined in an updated
  `gemini-extension.json` manifest file that describes their capabilities in a
  structured way.
- **Progressive Disclosure**: The model will be made aware of an extension's
  capabilities without being overwhelmed by the full content of the extension.
  The full details will be loaded on-demand.
- **Executable Tools**: Extensions will be able to include executable scripts
  that are exposed as tools to the model.
- **Discoverability**: Extensions will be discoverable and installable from
  various sources.

## 2. Updated `gemini-extension.json` Format

We will extend the `gemini-extension.json` manifest to support new, more
explicit properties for security and execution. This ensures backward
compatibility while allowing for a more robust and secure definition of an
extension's capabilities.

Here is an example of the updated `gemini-extension.json` format:

```json
{
  "name": "code-analyzer",
  "version": "1.0.0",
  "description": "An extension for analyzing and linting code files.",
  "permissions": {
    "filesystem": {
      "read": "workspace"
    },
    "network": {
      "allowedHosts": ["https://linter-rules.example.com"]
    }
  },
  "tools": [
    {
      "name": "lintFile",
      "description": "Lints a given file and returns a list of issues as a JSON object.",
      "executable": "tools/lint.py",
      "runtime": "python",
      "output": "json",
      "args": [
        {
          "name": "file",
          "type": "string",
          "description": "The path to the file to lint."
        }
      ]
    }
  ],
  "docs": [
    {
      "name": "linter-rules",
      "description": "A document explaining the linting rules used by the lintFile tool.",
      "file": "docs/rules.md"
    }
  ],
  "examples": [
    {
      "prompt": "Lint the file src/index.js",
      "tool_call": "lintFile(file=\"src/index.js\")"
    }
  ]
}
```

### New Manifest Properties

- **`permissions`**: An object that declares the security permissions required
  by the extension. This allows for granular control and provides transparency
  to the user.
  - `filesystem`: Defines file system access. `read` and `write` properties can
    be set to `none`, `workspace`, `home`, or `full`.
  - `network`: Defines network access. `allowedHosts` is an array of hostnames
    the extension can connect to. An empty array means no network access.
- **`runtime`**: (In `tools`) Specifies the runtime environment for the
  executable (e.g., `python`, `node`, `shell`). This removes ambiguity and
  ensures the script is executed correctly.
- **`output`**: (In `tools`) Defines the expected output format of the tool
  (`json` or `string`). This helps the CLI parse the tool's result reliably.
  Structured JSON output is preferred for model consumption.

## 3. Extension Directory Structure

An extension will be a directory with the following structure:

```
/my-extension
├── gemini-extension.json
├── tools/
│   └── lint.py
└── docs/
    └── rules.md
```

- `gemini-extension.json`: The manifest file.
- `tools/`: A directory containing executable scripts for the tools defined in
  the manifest.
- `docs/`: A directory for documentation and other context files.

## 4. Core Implementation Changes

### 4.1. `ExtensionManager` (`packages/cli/src/config/extension-manager.ts`)

The `ExtensionManager` will be updated to:

1.  **Parse Enhanced Manifest**: Enhance the parsing logic for
    `gemini-extension.json` to understand the new `tools`, `docs`, and
    `examples` properties.
2.  **Manage Capabilities**: The `ExtensionManager` will be responsible for
    managing the capabilities of each extension and providing them to the other
    parts of the CLI.

### 4.2. Progressive Disclosure within the `ExtensionManager`

The `ExtensionManager` will handle "progressive disclosure" of extension
capabilities:

- **Initial Context**: When a session starts, the `ExtensionManager` will
  provide a summary of the available extensions and their capabilities to the
  model. This summary will be generated from the `description` and the new
  properties in the `gemini-extension.json` files.
- **On-Demand Loading**: When the model decides to use a tool or access a
  document from an extension, the `ExtensionManager` will be responsible for
  loading the full content of the executable or the doc file into the context.

### 4.3. Prompt Engineering

The initial prompt sent to the Gemini model will be modified to include the list
of available extensions and their capabilities. For example:

```
You have the following extensions available:

- **code-analyzer**: An extension for analyzing and linting code files.
  - **Tools**: `lintFile(file: string)`
  - **Docs**: `linter-rules`

To use a tool, use the following format: <tool_code>...</tool_code>
To read a doc, use the following format: <read_doc>doc-name</read_doc>
```

## 5. Tool Execution

### 5.1. From Scripts to Tools

The CLI will need a mechanism to execute the scripts in the `tools` directory of
an extension and return the output to the model.

1.  When the model emits a `<tool_code>` block for an extension's tool, the CLI
    will identify the corresponding executable script from the
    `gemini-extension.json`.
2.  The CLI will execute the script in a sandboxed environment, configured
    according to the extension's `permissions`.
3.  Arguments provided by the model will be passed to the script as named flags
    (e.g., `script.py --argName 'value'`).
4.  The output of the script will be captured from `stdout` and returned to the
    model as the result of the tool call, parsed according to the tool's
    `output` type.

### 5.2. Security

Executing arbitrary scripts is a security risk. We will implement a
permissions-based security model to mitigate this, centered around the
`permissions` block in the `gemini-extension.json` manifest.

- **Permissions-Based Sandboxing**: All extension tools will be executed in a
  sandboxed environment that is configured based on the permissions declared in
  the manifest. For example, if an extension does not request network access,
  the sandbox will block all outgoing network calls.
- **Informed User Consent**: Before an extension's tool is executed for the
  first time, the user will be prompted for consent. The prompt will clearly
  state the permissions the extension is requesting (e.g., "The 'code-analyzer'
  extension is requesting read-only access to your workspace files. Allow?").
- **Auditing**: The `gemini extensions describe <extension-name>` command will
  display the full `permissions` block, allowing users to audit an extension's
  capabilities at any time.

## 6. User Experience

The existing `gemini extensions` command will be enhanced to support the new
capabilities:

- `gemini extensions install <git-repo>`: Install an extension from a Git
  repository.
- `gemini extensions uninstall <extension-name>`: Uninstall an extension.
- `gemini extensions list`: List all installed extensions.
- `gemini extensions describe <extension-name>`: This subcommand will be
  enhanced to show the new capabilities of an extension, including its tools,
  docs, and examples.

## 7. Backwards Compatibility

The proposed changes are designed to be backward compatible. The
`ExtensionManager` will be able to load both the old and new formats of the
`gemini-extension.json` file. Existing extensions that do not use the new
`tools`, `docs`, and `examples` properties will continue to work as they do now.

## 8. Example "Hello World" Extension

Here is an example of a simple "hello world" extension.

**Directory Structure:**

```
/hello-extension
├── gemini-extension.json
└── tools/
    └── say-hello.py
```

**`gemini-extension.json`:**

```json
{
  "name": "hello-extension",
  "version": "1.0.0",
  "description": "A simple extension that says hello.",
  "permissions": {
    "filesystem": {
      "read": "none",
      "write": "none"
    },
    "network": {
      "allowedHosts": []
    }
  },
  "tools": [
    {
      "name": "sayHello",
      "description": "Says hello to the given name.",
      "executable": "tools/say-hello.py",
      "runtime": "python",
      "args": [
        {
          "name": "name",
          "type": "string",
          "description": "The name to say hello to."
        }
      ]
    }
  ]
}
```

**`tools/say-hello.py`:**

```python
import argparse

if __name__ == "__main__":
  parser = argparse.ArgumentParser()
  parser.add_argument("--name", required=True, help="The name to say hello to.")
  args = parser.parse_args()
  print(f"Hello, {args.name}!")
```

## 9. Implementation Steps

This section provides a step-by-step guide to implementing the enhanced
extension capabilities.

### Step 1: Create a New Branch

Start by creating a new branch for this feature:

```bash
git checkout -b feature/enhanced-extensions
```

### Step 2: Update `gemini-extension.json` Schema

1.  **Define new types**: In `packages/cli/src/config/extension.ts`, define the
    interfaces for the new properties.

    ```typescript
    export type FilesystemAccess = 'none' | 'workspace' | 'home' | 'full';

    export interface ExtensionPermissions {
      filesystem?: {
        read?: FilesystemAccess;
        write?: FilesystemAccess;
      };
      network?: {
        allowedHosts?: string[];
      };
    }

    export interface ExtensionToolArg {
      name: string;
      type: string;
      description: string;
    }

    export interface ExtensionTool {
      name: string;
      description: string;
      executable: string;
      runtime: 'node' | 'python' | 'shell';
      output?: 'json' | 'string';
      args: ExtensionToolArg[];
    }

    export interface ExtensionDoc {
      name: string;
      description: string;
      file: string;
    }

    export interface ExtensionExample {
      prompt: string;
      tool_call: string;
    }
    ```

2.  **Update `ExtensionConfig`**: In the same file, add the new optional
    properties to the `ExtensionConfig` interface.

    ```typescript
    export interface ExtensionConfig {
      name: string;
      version: string;
      description?: string;
      permissions?: ExtensionPermissions;
      mcpServers?: { [key: string]: MCPServerConfig };
      contextFileName?: string | string[];
      excludeTools?: string[];
      tools?: ExtensionTool[];
      docs?: ExtensionDoc[];
      examples?: ExtensionExample[];
    }
    ```

### Step 3: Enhance `ExtensionManager`

1.  **Update `GeminiCLIExtension`**: In `@google/gemini-cli-core`, update the
    `GeminiCLIExtension` interface to include the new properties.

2.  **Parse new properties**: In `packages/cli/src/config/extension-manager.ts`,
    update the `loadExtension` function to parse the `permissions`, `tools`,
    `docs`, and `examples` properties from `gemini-extension.json` and store
    them in the `GeminiCLIExtension` object.

### Step 4: Implement Progressive Disclosure

1.  **Generate extension summary**: In `packages/cli/src/config/config.ts`,
    create a new function
    `generateExtensionSummary(extensions: GeminiCLIExtension[]): string`. This
    function will iterate over the loaded extensions and create a
    markdown-formatted string that summarizes their capabilities.

2.  **Include summary in prompt**: In the `loadCliConfig` function in the same
    file, call `generateExtensionSummary` and pass the result to the `Config`
    object. This will ensure the summary is included in the initial prompt.

### Step 5: Implement Tool Execution

1.  **Create `ExecutableToolService`**: Create a new file
    `packages/cli/src/services/ExecutableToolService.ts`. This service will be
    responsible for:
    - Taking a tool call from the model.
    - Finding the corresponding executable script from the loaded extensions.
    - Reading the extension's `permissions` to configure a secure, sandboxed
      environment (e.g., restricting network/filesystem access).
    - Using the `runtime` property to determine how to execute the script (e.g.,
      `node`, `python3`).
    - Translating the model's arguments into named command-line flags.
    - Executing the script and capturing its `stdout`, `stderr`, and exit code.
    - Parsing the `stdout` based on the tool's `output` type and returning the
      result.

2.  **Integrate with tool handling**: In the part of the code that handles tool
    calls (likely in `packages/cli/src/hooks/useGeminiStream.ts` or a related
    file), add logic to check if the tool call is for an executable tool from an
    extension. If it is, call the `ExecutableToolService` to execute it.

### Step 6: Update `gemini extensions describe`

1.  **Locate the command**: Find the implementation of the
    `gemini extensions describe` command (likely in
    `packages/cli/src/commands/extensions/describe.ts`).
2.  **Display new properties**: Update the command to display the `permissions`,
    `tools`, `docs`, and `examples` of the described extension, in addition to
    the existing information.

### Step 7: Testing

1.  **Unit Tests**:
    - Add unit tests for the new parsing logic in
      `packages/cli/src/config/extension.test.ts`.
    - Add unit tests for the `ExecutableToolService`, including tests for
      sandbox configuration and argument passing.
2.  **Integration Tests**:
    - Create a new integration test in the `integration-tests` directory to test
      the end-to-end flow of using an extension with executable tools.
3.  **End-to-End Tests**:
    - Add an e2e test that installs an extension with the new format and
      verifies that the `gemini extensions describe` command displays the
      correct information.
