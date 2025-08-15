
# CVCS CLI Documentation
CLI of Whole CodeBase Version Control Snapshot System 

---

## Table of Contents
1.  [Quick Start](#quick-start)
2.  [Global Configuration](#global-configuration)
3.  [Command Overview](#command-overview)
4.  [`cvcs init`](#cvcs-init)
5.  [`cvcs push`](#cvcs-push)
6.  [`cvcs pull`](#cvcs-pull)
7.  [`cvcs switch`](#cvcs-switch)
8.  [`cvcs branch`](#cvcs-branch)
9.  [`cvcs map`](#cvcs-map)
10. [`cvcs config`](#cvcs-config)
11. [`cvcs delete`](#cvcs-delete)
12. [`.cvcs/cvcs.yaml` File Structure](#cvcsyaml-file-structure)
13. [`.cvcsignore` File](#cvcsignore-file)
14. [Common Workflows](#common-workflows)
15. [Troubleshooting](#troubleshooting)

---

## Quick Start

If you're using CVCS for the first time, here's a simple getting started workflow:

```bash
# 1. Initialize CVCS repository in project root directory
$ cvcs init --name my-project --remote-url https://cvcs.example.com

# 2. Create first version snapshot
$ cvcs push -m "Initial commit" -v "v1.0.0"

# 3. View version history
$ cvcs map

# 4. Create new branch and push
$ cvcs branch -b feature/login -m "Add login feature" -v "v1.1.0-alpha"

# 5. Switch back to main branch
$ cvcs switch -b main
```

---

## Global Configuration

CVCS CLI supports the following environment variables for global configuration:

| Environment Variable | Description | Default Value |
| :--- | :--- | :--- |
| `CVCS_DEBUG` | Enable debug mode, output detailed logs | `false` |
| `CVCS_REMOTE_URL` | Default remote server URL | None |
| `CVCS_TIMEOUT` | API request timeout (seconds) | `30` |

### Configuration Examples
```bash
# Windows (PowerShell)
$env:CVCS_DEBUG="true"
$env:CVCS_REMOTE_URL="https://cvcs.example.com"

# macOS/Linux
export CVCS_DEBUG=true
export CVCS_REMOTE_URL=https://cvcs.example.com
```

---

## Command Overview

| Command | Purpose | Main Parameters |
| :--- | :--- | :--- |
| `init` | Initialize new repository | `--name`, `--remote-url` |
| `push` | Create version snapshot | `-m`, `-v`, `-b`, `--branch-from` |
| `pull` | Download and overwrite local files | `-b`, `-v`, `-y` |
| `switch` | Switch to specified version | `-b`, `-v`, `-y` |
| `branch` | Create new branch | `-b`, `-m`, `--branch-from`, `-v` |
| `map` | Display version history graph | `--json` |
| `config` | View or set configuration | `--mode`, `--storage-path`, `-l` |
| `delete` | Delete entire repository | `-y` |

---

## `cvcs init`

Initialize a new CVCS repository in the current directory.

### Usage
```bash
cvcs init [--name <name>] [--remote-url <url>] [--json]
```

### Description
This command sets up the current directory as a CVCS repository by creating a `.cvcs` subdirectory with a `cvcs.yaml` configuration file. It then registers this new codebase with the remote server.

### Options
| Option | Alias | Type | Description |
| :--- | :--- | :--- | :--- |
| `--name` | `-n` | `string` | Name of the codebase. Defaults to current directory name. |
| `--remote-url` | `-r` | `string` | URL of the remote CVCS server. If omitted, uses `CVCS_REMOTE_URL` environment variable. |
| `--json` | `-j` | `boolean`| Output result in JSON format. |

### Examples

#### Basic Initialization
```bash
$ cvcs init --name my-awesome-project --remote-url https://cvcs.example.com
```
**Expected Response:**
```
✔ Initialized CVCS repository.
✔ Created .cvcs/cvcs.yaml
✔ Registered codebase 'my-awesome-project' with remote.
Codebase ID: c1b2a3d4-e5f6-7890-1234-567890abcdef
```

#### Simplified Initialization Using Environment Variable
```bash
$ export CVCS_REMOTE_URL=https://cvcs.example.com
$ cvcs init --name my-project
```

#### JSON Output
```bash
$ cvcs init --name my-awesome-project --remote-url https://cvcs.example.com --json
```
**Expected Response:**
```json
{
  "codebase": {
    "id": "c1b2a3d4-e5f6-7890-1234-567890abcdef",
    "name": "my-awesome-project",
    "description": null,
    "branch": "main",
    "created_at": "2023-10-27T10:00:00Z",
    "updated_at": "2023-10-27T10:00:00Z"
  }
}
```

### Response Format Definition (`init`)
*   `codebase`: (object) Information about the created codebase.
    *   `id`: (string, UUID) Unique identifier of the codebase.
    *   `name`: (string) Name of the codebase.
    *   `description`: (string|null) Description of the codebase.
    *   `branch`: (string) Name of the default main branch, usually `main`.
    *   `created_at`: (string, ISO 8601) Creation timestamp.
    *   `updated_at`: (string, ISO 8601) Last update timestamp.

### Error Handling
*   **Directory already is CVCS repository**: If current directory already contains `.cvcs` directory, command will fail.
*   **Network connection failure**: If unable to connect to remote server, command will fail and retain local `.cvcs` directory.
*   **Name conflict**: If a codebase with the same name already exists on remote, command will fail.

---

## `cvcs push`

Create and upload a new version snapshot to the remote repository.

### Usage
```bash
cvcs push -m <message> [-v <version>] [-b <branch>] [--branch-from <source>] [--json]
```

### Description
The `push` command scans file changes in the current workspace (following `.cvcsignore` rules), packages them into a snapshot, and uploads to the remote server. Each push creates a new version. This command also updates the local `cvcs.yaml` with the latest version map returned from the server.

### Options
| Option | Alias | Type | Description |
| :--- | :--- | :--- | :--- |
| `--message` | `-m` | `string` | **Required.** Provide descriptive information for the version snapshot. |
| `--version` | `-v` | `string` | Semantic version number (e.g., "v1.0.1"). If omitted, a UUID will be generated. |
| `--branch` | `-b` | `string` | Branch to push to. Defaults to current branch in `cvcs.yaml`. |
| `--branch-from` | | `string` | Create new branch from a specified version, format: `<branch>@<version>` or `<version-id>`. |
| `--json` | `-j` | `boolean`| Output result in JSON format. |

### Examples

#### Push to Current Branch
```bash
$ cvcs push . -m "Implemented feature X" -v "v1.1.0"
```
**Expected Response:**
```
✔ Scanned 15 files.
✔ Created snapshot for version v1.1.0.
✔ Uploading snapshot... (1.23 MB)
✔ Snapshot uploaded successfully.
✔ Local configuration updated.

Successfully pushed version v1.1.0 to branch main.
```

#### Create New Branch from Existing Version
```bash
$ cvcs push . -m "Start work on new feature" -v "v2.0.0-alpha" -b "feature/new-login" --branch-from "main@v1.1.0"
```
**Expected Response:**
```
✔ Scanned 20 files.
✔ Created snapshot for version v2.0.0-alpha.
✔ Uploading snapshot... (1.5 MB)
✔ Snapshot uploaded successfully.
✔ Local configuration updated.

Successfully created branch feature/new-login and pushed version v2.0.0-alpha.
```

#### Create Branch Using UUID as Version ID
```bash
$ cvcs push . -m "Hotfix based on specific commit" -b "hotfix/urgent" --branch-from "a1b2c3d4-e5f6-7890-1234-567890abcdef"
```

#### JSON Output
```bash
$ cvcs push . -m "Implemented feature X" -v "v1.1.0" --json
```
**Expected Response:**
```json
{
  "codebase": {
    "id": "c1b2a3d4-e5f6-7890-1234-567890abcdef",
    "name": "my-awesome-project",
    "updated_at": "2023-10-27T11:05:00Z"
  },
  "version": {
    "id": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
    "version": "v1.1.0",
    "branch": "main",
    "message": "Implemented feature X",
    "created_at": "2023-10-27T11:05:00Z",
    "stats": {
      "total_files": 15,
      "total_size": 1258291,
      "compressed_size": 314572,
      "compression_ratio": 0.25
    }
  },
  "file_tree": {
    "tree_id": "t1b2c3d4-e5f6-7890-1234-567890abcdef",
    "version_id": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
    "files": [
      {
        "path": "src/index.ts",
        "hash": "sha256-hash-string",
        "size": 1200,
        "compressed_size": 400,
        "oss_key": "my-awesome-project/src/index.ts",
        "type": "other"
      }
    ],
    "generated_at": "2023-10-27T11:04:59Z"
  },
  "version_map": {
    "codebase_id": "c1b2a3d4-e5f6-7890-1234-567890abcdef",
    "nodes": [
        { "id": "a1b2c3d4-...", "version": "v1.1.0", "branch": "main", "message": "...", "created_at": "...", "stats": { "...": "..." } },
        { "id": "z9y8x7w6-...", "version": "v1.0.0", "branch": "main", "message": "...", "created_at": "...", "stats": { "...": "..." } }
    ],
    "edges": [
        { "from": "a1b2c3d4-...", "to": "z9y8x7w6-...", "linkage_type": "sequential" }
    ],
    "refs": {
        "main": "a1b2c3d4-..."
    }
  }
}
```

### Response Format Definition (`push` & `branch`)
*   `codebase`: (object) Updated codebase information.
*   `version`: (object) Newly created version information.
    *   `id`: (string, UUID) Unique identifier of the new version.
    *   `stats`: (object) Statistics about this version snapshot.
        *   `total_files`: (integer) Total number of files.
        *   `total_size`: (integer) Total original file size (bytes).
        *   `compressed_size`: (integer) Total compressed size (bytes).
        *   `compression_ratio`: (float) Compression ratio.
*   `file_tree`: (object) File tree details for this snapshot.
    *   `files`: (array) List of file entries, each containing `path`, `hash`, `size`, etc.
*   `version_map`: (object) **Complete, updated** version history graph, including `nodes`, `edges`, and `refs`. Structure is the same as [`cvcs map --json`](#cvcs-map) output.

### Error Handling
*   **No file changes**: If workspace has no files, push will fail.
*   **Version number conflict**: If specified version number already exists in the branch, push will fail.
*   **Source version doesn't exist**: If version specified by `--branch-from` doesn't exist, push will fail.

---

## `cvcs pull`

Download snapshot from remote and overwrite local tracked files.

### Usage
```bash
cvcs pull [-b <branch>] [-v <version>] [-y] [--json]
```

### Description
The `pull` command fetches a specified version snapshot from the remote repository. It first clears all local tracked files, then extracts the downloaded snapshot content to the workspace. Files and directories listed in `.cvcsignore` are not affected. After successful pull, the `header` in `cvcs.yaml` is updated to point to the new version, but the version `map` remains unchanged.

### Options
| Option | Alias | Type | Description |
| :--- | :--- | :--- | :--- |
| `--branch` | `-b` | `string` | Branch to pull from. Defaults to current branch. |
| `--ver` | `-v` | `string` | Version to pull. Can be semantic version number or UUID. Defaults to latest version of specified branch. |
| `--yes` | `-y` | `boolean`| Skip interactive confirmation prompt. |
| `--json` | `-j` | `boolean`| Output final version node information in JSON format. |

### Examples

#### Interactive Mode
Running `cvcs pull` without branch or version parameters enters interactive mode.
```bash
$ cvcs pull
```
**Expected Response:**
```
Version map for context:
📦 my-awesome-project
  🏷️   v1.0.0       ✨ "Initial commit"
  🏷️   v1.1.0       ✨ "Implemented feature X" (HEAD -> main)
  └── 🌿 feature/new-login (from v1.1.0)
        🏷️   v2.0.0-alpha   ✨ "Start work on new feature" (HEAD -> feature/new-login)

? Select a version to pull (Ctrl+C to cancel): (Use arrow keys)
❯ [main      ] v1.1.0       - Implemented feature X (1h ago)
  [main      ] v1.0.0       - Initial commit (2h ago)
  [feature/new-login] v2.0.0-alpha - Start work on new feature (30m ago)

This will overwrite local tracked files to match version v1.1.0. Ignored files will be preserved. Continue? (y/N) y
✔ Cleared local tracked files.
✔ Downloaded version v1.1.0 (1.23 MB)
✔ Extracted new version successfully.
✔ Local configuration updated.

Successfully pulled and switched to main @ v1.1.0
```

#### Non-interactive (Parameterized) Pull
```bash
$ cvcs pull -b main -v v1.0.0 -y
```
**Expected Response:**
```
✔ Cleared local tracked files.
✔ Downloaded version v1.0.0 (950 KB)
✔ Extracted new version successfully.
✔ Local configuration updated.

Successfully pulled and switched to main @ v1.0.0
```

#### Pull Specific Version Using UUID
```bash
$ cvcs pull -v a1b2c3d4-e5f6-7890-1234-567890abcdef -y
```

#### JSON Output
```bash
$ cvcs pull -b main -v v1.0.0 -y --json
```
**Expected Response:**
```json
{
  "id": "z9y8x7w6-...",
  "version": "v1.0.0",
  "branch": "main",
  "message": "Initial commit",
  "created_at": "2023-10-27T10:00:00Z",
  "stats": {
    "total_files": 10,
    "total_size": 900000,
    "compressed_size": 200000,
    "compression_ratio": 0.22
  }
}
```

### Response Format Definition (`pull` & `switch`)
*   **Version Node Object**: JSON output is the node information of the target version.
    *   `id`: (string, UUID) Unique identifier of the version.
    *   `version`: (string) Semantic version number.
    *   `branch`: (string) Branch it belongs to.
    *   `message`: (string) Commit message.
    *   `created_at`: (string, ISO 8601) Creation timestamp.
    *   `stats`: (object) Statistics for this version.

### Error Handling
*   **Version doesn't exist**: If specified version doesn't exist in local graph, pull will fail.
*   **Network error**: If unable to download files from remote, pull will fail and local files remain unchanged.
*   **Local file permissions**: If unable to delete or write local files, pull will partially fail.

---

## `cvcs switch`

Switch local workspace to another version by downloading files.

### Usage
```bash
cvcs switch [-b <branch>] [-v <version>] [-y] [--json]
```

### Description
The `switch` command is functionally similar to `pull`. It allows you to switch the local workspace to another version state recorded in the local `cvcs.yaml` graph. It clears existing tracked files and downloads the target version files from remote.

### Options
| Option | Alias | Type | Description |
| :--- | :--- | :--- | :--- |
| `--branch` | `-b` | `string` | Target branch to switch to. |
| `--ver` | `-v` | `string` | Target version to switch to. Defaults to latest version of the branch. |
| `--yes` | `-y` | `boolean`| Skip interactive confirmation prompt. |
| `--json` | `-j` | `boolean`| Output final version node information in JSON format. |

### Examples
Examples for `cvcs switch` are identical in behavior and output to [`cvcs pull`](#cvcs-pull). It also supports both interactive and non-interactive modes.

#### Switch to Latest Feature Branch
```bash
$ cvcs switch -b feature/new-login -y
```

#### Switch to Specific Version of Main Branch
```bash
$ cvcs switch -b main -v v1.0.0 -y
```

---

## `cvcs branch`

A convenience command for creating new versions on new branches.

### Usage
```bash
cvcs branch -b <new-branch-name> -m <message> [--branch-from <source>] [-v <version>] [--json]
```

### Description
This command is a specialized wrapper around `cvcs push`, designed to make creating new branches more explicit. It requires you to provide a new branch name and a message for the first commit on that branch.

### Options
| Option | Alias | Type | Description |
| :--- | :--- | :--- | :--- |
| `--branch` | `-b` | `string` | **Required.** Name of the new branch to create. |
| `--message` | `-m` | `string` | **Required.** Descriptive information for the new version. |
| `--branch-from` | | `string` | Source version to create branch from, e.g., `main@v1.1.0`. Defaults to current HEAD. |
| `--version` | `-v` | `string` | Semantic version for the new snapshot. If omitted, a UUID will be generated. |
| `--json` | `-j` | `boolean`| Output result in JSON format. |

### Examples

#### Create New Feature Branch
```bash
$ cvcs branch -b hotfix/typo-fix -m "Fix typo in README" --branch-from main@v1.1.0 -v v1.1.1
```
**Expected Response:**
```
✔ Scanned 15 files.
✔ Created snapshot for version v1.1.1.
✔ Uploading snapshot... (1.23 MB)
✔ Snapshot uploaded successfully.
✔ Local configuration updated.

Successfully created branch hotfix/typo-fix and pushed version v1.1.1.
```

#### Create Feature Branch from Current HEAD
```bash
$ cvcs branch -b feature/api-improvement -m "Start API refactoring" -v v2.0.0-alpha
```

#### Create Branch Based on Specific UUID
```bash
$ cvcs branch -b experimental/new-algo -m "Test new algorithm" --branch-from a1b2c3d4-e5f6-7890-1234-567890abcdef
```

### Response Format Definition (`branch`)
The JSON response format for `cvcs branch` is identical to [`cvcs push`](#response-format-definition-push--branch).

### Error Handling
*   **Branch name already exists**: If specified branch name already exists, branch will fail.
*   **Source version doesn't exist**: If version specified by `--branch-from` doesn't exist, branch will fail.

---

## `cvcs map`

Display version history graph based on local `cvcs.yaml` file.

### Usage
```bash
cvcs map [--json]
```

### Description
This command reads version history (`nodes` and `edges`) from the local `cvcs.yaml` file and renders a visual, tree-like graph in the terminal. This allows you to view relationships between different versions and branches without connecting to the remote server.

### Options
| Option | Alias | Type | Description |
| :--- | :--- | :--- | :--- |
| `--json` | `-j` | `boolean`| Output raw graph data in JSON format. |

### Examples

#### Standard Graph View
```bash
$ cvcs map
```
**Expected Response:**
```
📦 my-awesome-project
  🏷️   v1.0.0       ✨ "Initial commit"
  🏷️   v1.1.0       ✨ "Implemented feature X" (HEAD -> main)
  └── 🌿 hotfix/typo-fix (from v1.1.0)
        🏷️   v1.1.1   ✨ "Fix typo in README" (HEAD -> hotfix/typo-fix)
```

#### Complex Branch Structure Example
```
📦 my-complex-project
  🏷️   v1.0.0       ✨ "Initial release"
  🏷️   v1.1.0       ✨ "Add user system"
  🏷️   v1.2.0       ✨ "Performance improvements" (HEAD -> main)
  ├── 🌿 feature/api-v2 (from v1.1.0)
  │   🏷️   v2.0.0-alpha   ✨ "Start API v2 development"
  │   🏷️   v2.0.0-beta    ✨ "Complete API v2 core" (HEAD -> feature/api-v2)
  └── 🌿 hotfix/security (from v1.2.0)
        🏷️   v1.2.1   ✨ "Security patch" (HEAD -> hotfix/security)
```

#### JSON Output
```bash
$ cvcs map --json
```
**Expected Response:**
```json
{
  "codebase_id": "c1b2a3d4-e5f6-7890-1234-567890abcdef",
  "nodes": [
    {
      "id": "uuid-1",
      "version": "v1.0.0",
      "branch": "main",
      "message": "Initial commit",
      "created_at": "2023-10-27T10:00:00Z",
      "stats": {
        "total_files": 10,
        "total_size": 900000,
        "compressed_size": 200000,
        "compression_ratio": 0.22
      }
    },
    {
      "id": "uuid-2",
      "version": "v1.1.0",
      "branch": "main",
      "message": "Implemented feature X",
      "created_at": "2023-10-27T11:00:00Z",
      "stats": {
        "total_files": 15,
        "total_size": 1258291,
        "compressed_size": 314572,
        "compression_ratio": 0.25
      }
    },
    {
      "id": "uuid-3",
      "version": "v1.1.1",
      "branch": "hotfix/typo-fix",
      "message": "Fix typo in README",
      "created_at": "2023-10-27T12:00:00Z",
      "stats": {
        "total_files": 15,
        "total_size": 1258345,
        "compressed_size": 314580,
        "compression_ratio": 0.25
      }
    }
  ],
  "edges": [
    {
      "from": "uuid-2",
      "to": "uuid-1",
      "linkage_type": "sequential"
    },
    {
      "from": "uuid-3",
      "to": "uuid-2",
      "linkage_type": "branch_from"
    }
  ],
  "refs": {
    "main": "uuid-2",
    "hotfix/typo-fix": "uuid-3"
  }
}
```

### Response Format Definition (`map`)
*   `codebase_id`: (string, UUID) Unique identifier of the codebase.
*   `nodes`: (array) List of version nodes. Each node object contains:
    *   `id`: (string, UUID) Unique identifier of the version.
    *   `version`: (string) Semantic version number.
    *   `branch`: (string) Branch it belongs to.
    *   `message`: (string) Commit message.
    *   `created_at`: (string, ISO 8601) Creation timestamp.
    *   `stats`: (object) Version statistics.
*   `edges`: (array) List of connections between versions.
    *   `from`: (string, UUID) Child node ID (newer version).
    *   `to`: (string, UUID) Parent node ID (older version).
    *   `linkage_type`: (string) Link type:
        *   `sequential`: Consecutive versions within the same branch
        *   `branch_from`: Cross-branch creation relationship
*   `refs`: (object) Branch reference mapping, keys are branch names, values are node IDs of the latest version of that branch.

---

## `cvcs config`

View or set configuration options for the local repository.

### Usage
`cvcs config [--mode <mode>] [--storage-path <path>] [-l] [--json]`

### Description
The `config` command allows you to manage configuration in the local `.cvcs/cvcs.yaml` file. It's mainly used for switching between different backend modes (cloud vs. local) and specifying file storage paths in local mode. When run without any parameters, it lists the current configuration.

### Options
| Option | Alias | Type | Description |
| :--- | :--- | :--- | :--- |
| `--mode` | | `string` | Set API mode. Options are `cloud` or `local`. |
| `--storage-path` | | `string` | **Only available in local mode.** Set local CVCS server file storage path. Supports absolute and relative paths. |
| `--list` | `-l` | `boolean`| List all current configuration. This is the default behavior. |
| `--json` | `-j` | `boolean`| Output result in JSON format (only for `list`). |

### Examples

#### View Current Configuration
```bash
$ cvcs config
# or
$ cvcs config -l
```
**Expected Response (Cloud Mode):**
```
Current configuration:
- API Mode: cloud
- API Endpoint: https://cvcs-dbeekchwgb.us-west-1.fcapp.run/api/v1
- Codebase ID: c1b2a3d4-e5f6-7890-1234-567890abcdef
```
**Expected Response (Local Mode):**
```
Current configuration:
- API Mode: local
- API Endpoint: http://localhost:8080/api/v1
- Local Storage Path: D:\cvcs_storage
- Codebase ID: c1b2a3d4-e5f6-7890-1234-567890abcdef
```

#### Switch to Local Mode
```bash
$ cvcs config --mode local
```
**Expected Response:**
```
Successfully set API mode to: local
API endpoint is now: http://localhost:8080/api/v1
```

#### Set Storage Path in Local Mode
```bash
$ cvcs config --storage-path ./my_local_storage
```
**Expected Response:**
```
Successfully set local server storage path to: /path/to/your/project/my_local_storage
```

#### Set Mode and Path Simultaneously
```bash
$ cvcs config --mode local --storage-path "D:\cvcs-data"
```
**Expected Response:**
```
Successfully set API mode to: local
API endpoint is now: http://localhost:8080/api/v1
Successfully set local server storage path to: D:\cvcs-data
```

### Error Handling
*   **Mode error**: If attempting to set `--storage-path` in `cloud` mode, command will fail with an error message.
*   **Not initialized**: If run in a non-CVCS repository, command will fail.

---

## `cvcs delete`

Permanently delete codebase from remote and remove local repository.

### Usage
```bash
cvcs delete [-y] [--json]
```

### Description
**This is a destructive, irreversible operation.** The `delete` command connects to the remote server to permanently remove the entire codebase, including all its version history. To prevent accidental deletion, it requires you to enter the codebase name for confirmation. After successful remote deletion, it also removes the local `.cvcs` directory.

### Options
| Option | Alias | Type | Description |
| :--- | :--- | :--- | :--- |
| `--yes` | `-y` | `boolean`| **Dangerous.** Skip confirmation prompt. |
| `--json` | `-j` | `boolean`| Output result in JSON format. |

### Examples

#### Interactive Deletion (Safe)
```bash
$ cvcs delete
```
**Expected Response:**
```
! DANGER !
You are about to permanently delete the codebase 'my-awesome-project' (ID: c1b2a3d4-...).
This action is irreversible and will delete all versions from the remote.

? To confirm, please type the codebase name ('my-awesome-project'): my-awesome-project
✔ Deleting codebase 'my-awesome-project' from the remote...
✔ Successfully deleted codebase 'my-awesome-project' from the remote.
✔ Removing local .cvcs directory...
✔ Successfully removed local .cvcs directory.

Codebase 'my-awesome-project' has been completely deleted.
```

#### Force Deletion (Unsafe)
```bash
$ cvcs delete --yes
```
**Expected Response:**
```
! DANGER !
You are about to permanently delete the codebase 'my-awesome-project' (ID: c1b2a3d4-...).
This action is irreversible and will delete all versions from the remote.

Skipping confirmation due to --yes flag. Proceeding with deletion...
✔ Deleting codebase 'my-awesome-project' from the remote...
✔ Successfully deleted codebase 'my-awesome-project' from the remote.
✔ Removing local .cvcs directory...
✔ Successfully removed local .cvcs directory.

Codebase 'my-awesome-project' has been completely deleted.
```

#### Confirmation Failure
```bash
$ cvcs delete
```
**Expected Response:**
```
...
? To confirm, please type the codebase name ('my-awesome-project'): wrong-name
✖ Confirmation failed. Aborting delete operation.
The remote deletion might have failed. For safety, the local .cvcs directory has NOT been removed.
```

#### JSON Output
```bash
$ cvcs delete --yes --json
```
**Expected Response:**
```json
{
  "message": "Codebase 'my-awesome-project' and local .cvcs directory deleted successfully.",
  "codebaseId": "c1b2a3d4-e5f6-7890-1234-567890abcdef"
}
```

### Response Format Definition (`delete`)
*   `message`: (string) Success message describing the operation result.
*   `codebaseId`: (string, UUID) ID of the deleted codebase.

### Error Handling
*   **Remote deletion failure**: If remote deletion fails, local `.cvcs` directory will be preserved.
*   **Local deletion failure**: If remote deletion succeeds but local deletion fails, manual deletion will be prompted.

---

## `.cvcs/cvcs.yaml` File Structure

`cvcs.yaml` is the core configuration file for CVCS, stored in the `.cvcs` folder in the repository root. It contains all metadata for the local repository, including project information, remote configuration, and complete version history.

### Complete Example
```yaml
cvcs_version: '1.0'
codebase:
  id: c1b2a3d4-e5f6-7890-1234-567890abcdef
  name: my-awesome-project
remote:
  url: https://cvcs.example.com
header:
  id: a1b2c3d4-e5f6-7890-1234-567890abcdef
  branch: main
  file_tree:
    tree_id: t1b2c3d4-e5f6-7890-1234-567890abcdef
    version_id: a1b2c3d4-e5f6-7890-1234-567890abcdef
    tree:
      src:
        index.ts: index.ts
        utils:
          helper.ts: helper.ts
      package.json: package.json
      README.md: README.md
    generated_at: '2023-10-27T11:05:00Z'
  map:
    codebase_id: c1b2a3d4-e5f6-7890-1234-567890abcdef
    nodes:
      - id: a1b2c3d4-e5f6-7890-1234-567890abcdef
        version: v1.1.0
        branch: main
        message: Implemented feature X
        created_at: '2023-10-27T11:05:00Z'
        stats:
          total_files: 15
          total_size: 1258291
          compressed_size: 314572
          compression_ratio: 0.25
      - id: z9y8x7w6-v5u4-3210-fedc-ba9876543210
        version: v1.0.0
        branch: main
        message: Initial commit
        created_at: '2023-10-27T10:00:00Z'
        stats:
          total_files: 10
          total_size: 900000
          compressed_size: 200000
          compression_ratio: 0.22
    edges:
      - from: a1b2c3d4-e5f6-7890-1234-567890abcdef
        to: z9y8x7w6-v5u4-3210-fedc-ba9876543210
        linkage_type: sequential
    refs:
      main: a1b2c3d4-e5f6-7890-1234-567890abcdef
```

### Structure Definition

#### Top-level Fields
| Field | Type | Description |
| :--- | :--- | :--- |
| `cvcs_version` | `string` | Version number of CVCS configuration file, used for forward compatibility. |
| `codebase` | `object` | Contains global information about the codebase. |
| `remote` | `object` | Remote server configuration. |
| `header` | `object` | "Pointer" and cache describing the current state of local workspace. |

#### `codebase` Object
| Field | Type | Description |
| :--- | :--- | :--- |
| `id` | `string` | Unique UUID of the codebase obtained from remote. |
| `name` | `string` | Name of the codebase. |

#### `remote` Object
| Field | Type | Description |
| :--- | :--- | :--- |
| `url` | `string` | Root URL of the remote CVCS API server. |

#### `header` Object
`header` contains all dynamic information about the local workspace.
| Field | Type | Description |
| :--- | :--- | :--- |
| `id` | `string` | UUID of the version snapshot corresponding to current workspace (i.e., HEAD). |
| `branch` | `string` | Name of the current branch. |
| `file_tree`| `object` | Complete file structure tree of current version. |
| `map` | `object` | Locally cached complete version history graph. |

#### `header.file_tree` Object
| Field | Type | Description |
| :--- | :--- | :--- |
| `tree_id` | `string` | Unique UUID of the file tree on remote. |
| `version_id`| `string` | UUID of the version this file tree belongs to. |
| `tree` | `object` | A nested object representing directory structure. Keys are file or directory names, values are either relative paths to the file (for files) or another nested `tree` object (for directories). |
| `generated_at`| `string` | Timestamp when this file tree was generated. |

#### `header.map` Object
This object has the same structure as the output of `cvcs map --json`.
| Field | Type | Description |
| :--- | :--- | :--- |
| `codebase_id` | `string` | UUID of the codebase. |
| `nodes` | `array` | Array of version node objects. |
| `edges` | `array` | Array of version connection relationship objects. |
| `refs` | `object`| Branch reference mapping, mapping branch names to their latest version IDs. |

---

## `.cvcsignore` File

The `.cvcsignore` file is used to specify files and directories that CVCS should ignore. Ignored files are not included in version snapshots and are not deleted by `pull` or `switch` commands.

### Syntax Rules
*   Each line specifies one ignore pattern
*   Lines starting with `#` are comments
*   Supports glob pattern matching
*   Patterns starting with `/` match from repository root
*   Patterns ending with `/` only match directories
*   Use `!` prefix to negate ignoring

### Example
```gitignore
# Dependency directories
node_modules/
vendor/

# Build output
dist/
build/
*.o
*.so

# Log files
*.log
logs/

# Environment configuration
.env
.env.local

# IDE files
.vscode/
.idea/
*.swp

# System files
.DS_Store
Thumbs.db

# But include specific configuration files
!.env.example

# Only ignore temp folder in root directory
/temp/

# Ignore all .tmp files except important.tmp
*.tmp
!important.tmp
```

### Default Ignore Rules
CVCS has the following built-in default ignore rules:
*   `.cvcs/` - CVCS own configuration directory
*   `.git/` - Git configuration directory
*   `node_modules/` - Node.js dependencies
*   `*.log` - Log files
*   `.DS_Store` - macOS system files

---

## Common Workflows

### Workflow 1: Basic Development Flow
```bash
# 1. Initialize project
$ cvcs init --name my-project

# 2. Create first version
$ cvcs push -m "Initial commit" -v "v1.0.0"

# 3. Develop new features
# ... write code ...

# 4. Push new version
$ cvcs push -m "Add user authentication" -v "v1.1.0"

# 5. View history
$ cvcs map
```

### Workflow 2: Feature Branch Development
```bash
# 1. Create feature branch from main branch
$ cvcs branch -b feature/payment -m "Start payment system" --branch-from main@v1.1.0

# 2. Continue development on feature branch
$ cvcs push -m "Add payment API" -v "v1.2.0-alpha"

# 3. Switch back to main branch
$ cvcs switch -b main

# 4. Switch back to feature branch to continue development
$ cvcs switch -b feature/payment

# 5. Complete feature development
$ cvcs push -m "Complete payment system" -v "v1.2.0"
```

### Workflow 3: Hotfix Process
```bash
# 1. Create hotfix branch from production version
$ cvcs branch -b hotfix/security-patch -m "Critical security fix" --branch-from main@v1.1.0

# 2. Apply fix
$ cvcs push -m "Fix security vulnerability" -v "v1.1.1"

# 3. View branch structure
$ cvcs map

# 4. After deployment, can delete branch content (by switching to other branches)
```

### Workflow 4: Version Rollback
```bash
# 1. View version history
$ cvcs map

# 2. Rollback to stable version
$ cvcs switch -v "v1.0.0" -y

# 3. Create fix branch based on stable version
$ cvcs branch -b fix/rollback-issue -m "Fix issues from rollback" --branch-from main@v1.0.0
```

---

## Troubleshooting

### Common Issues

#### 1. Unable to Connect to Remote Server
**Problem**: `Error: Unable to connect to remote server`
**Solutions**:
*   Check network connection
*   Verify remote URL is correct
*   Check firewall settings
*   Use environment variable `CVCS_DEBUG=true` to view detailed error information

#### 2. Version Conflict
**Problem**: `Error: Version 'v1.1.0' already exists in branch 'main'`
**Solutions**:
*   Use a different version number
*   Or omit `-v` parameter to let system auto-generate UUID

#### 3. Local Files in Use
**Problem**: `Error: Cannot delete file: permission denied`
**Solutions**:
*   Close programs using the files
*   Check file permissions
*   Run as administrator on Windows

#### 4. Snapshot Too Large
**Problem**: `Warning: Snapshot size exceeds recommended limit`
**Solutions**:
*   Check and update `.cvcsignore` file
*   Exclude large binary files
*   Consider using external storage services

#### 5. `.cvcs` Directory Corrupted
**Problem**: `Error: Invalid cvcs.yaml format`
**Solutions**:
*   Check YAML syntax in `cvcs.yaml` file
*   Re-pull configuration from remote: `cvcs pull -y`
*   Last resort: delete `.cvcs` directory and re-run `cvcs init`

### Debug Commands
```bash
# Enable verbose logging
$ export CVCS_DEBUG=true
$ cvcs push -m "test"

# Check configuration file
$ cat .cvcs/cvcs.yaml

# Verify ignore rules
$ cvcs push -m "test" --dry-run  # (if supported)
```

### Getting Help
```bash
# View command help
$ cvcs --help
$ cvcs push --help

# View version information
$ cvcs --version
```
