---
name: code-interpreter-execute
description: Execute code (Python, JS, TS) and shell commands in a secure AWS AgentCore Code Interpreter sandbox. Use for AWS troubleshooting, querying logs, or running code in isolation.
allowed-tools: Bash(agentcore-sandbox-*)
user-invocable: true
disable-model-invocation: true
argument-hint: "<code or description of what to run>"
---

You have access to `agentcore-sandbox`, a CLI tool that executes code and shell
commands in an isolated AWS AgentCore Code Interpreter sandbox.

## How to Handle Requests

Execute exactly what the user asks for — no more, no less. If the user says
"list my S3 buckets", run exactly that command. Don't add extra analysis,
formatting, or follow-up queries they didn't request.

If the request is ambiguous or missing key details, ask clarification questions
before executing anything. For example:

- **Unclear scope**: "Query my logs" → Which log group? What time range? What are you looking for?
- **Missing resource**: "Check my DynamoDB table" → Which table name? Which region?
- **Ambiguous language**: "Run this code" with no code provided → What code would you like to run? In which language?
- **Risky operations**: "Delete items from the table" → Which items? Can you confirm the filter criteria?

The goal is to get it right the first time rather than guess and run the wrong
thing. Once you're confident you understand the request, execute it immediately
without unnecessary preamble.

## First: Detect Platform

Before running any commands, detect the platform to select the correct binary.
Follow the instructions in `references/platform-detection.md`.

Store the resolved binary name (e.g., `agentcore-sandbox-darwin-arm64`) and use
it for all commands below.

## Then: Set AWS_REGION

Before invoking any `agentcore-sandbox` command, resolve the target region on
the **host** and store it in a shell variable:

```bash
export AWS_REGION="${CUSTOM_AGENTCORE_AWS_REGION:-$AWS_REGION}"
```

This prefers `CUSTOM_AGENTCORE_AWS_REGION` when set, and falls back to the
existing `AWS_REGION` value otherwise. Run this once at the start of every
skill invocation (i.e., in the same shell block where you detect the platform),
before any `run`, `start`, `exec`, or `stop` commands.

**Important:** The sandbox environment does NOT automatically inherit the host's
environment variables. You must explicitly pass the region into every sandbox
invocation. Each SDK also reads a different env var:

| SDK | Env var it reads |
|-----|-----------------|
| **boto3 / botocore (Python)** | `AWS_DEFAULT_REGION` (NOT `AWS_REGION`) |
| **AWS SDK for JavaScript/TypeScript** | `AWS_REGION` |
| **AWS CLI** | `AWS_DEFAULT_REGION` (or `AWS_REGION` as fallback) |

To avoid mistakes, **set both `AWS_DEFAULT_REGION` and `AWS_REGION`** in every
sandbox invocation so all SDKs and the AWS CLI work correctly regardless of
language:

- **Python**: Prepend `import os; os.environ['AWS_DEFAULT_REGION'] = os.environ['AWS_REGION'] = '<region>';` before any `import boto3` or SDK usage.
- **JavaScript/TypeScript**: Prepend `process.env.AWS_REGION = '<region>';` at the top of the code.
- **Shell (`--cmd`)**: Prepend `export AWS_DEFAULT_REGION='<region>' AWS_REGION='<region>';` before the command.

For `run` and `exec`, substitute the host `$AWS_REGION` variable into the code
string:

```bash
$BINARY run "import os; os.environ['AWS_DEFAULT_REGION'] = os.environ['AWS_REGION'] = '$AWS_REGION'; <rest of code>"
$BINARY run --lang js "process.env.AWS_REGION = '$AWS_REGION'; <rest of code>"
$BINARY run --cmd "export AWS_DEFAULT_REGION='$AWS_REGION' AWS_REGION='$AWS_REGION'; <rest of command>"
```

## Commands

| Command | Description |
|---------|-------------|
| `run [flags] <code>` | One-shot: start a session, execute, then stop |
| `start [--timeout N]` | Start a new session, prints session ID |
| `exec <session-id> [flags] <code>` | Execute code in an existing session |
| `stop <session-id>` | Stop an existing session |

### Flags

| Flag | Values | Default | Applies to |
|------|--------|---------|------------|
| `--lang` | `python`, `javascript`/`js`, `typescript`/`ts` | `python` | `exec`, `run` |
| `--cmd` | *(flag, no value)* | | `exec`, `run` |
| `--runtime` | `nodejs`, `deno`, `python` | auto | `exec`, `run` |
| `--clear-context` | *(flag)* | | `exec`, `run` |
| `--json` | *(flag)* | | `start`, `exec`, `run` |
| `--timeout` | seconds | `900` | `start`, `run` |

- `--lang` and `--cmd` are mutually exclusive. Use `--cmd` for shell commands, `--lang` for code execution.
- When `--runtime` is omitted, it defaults to `python` for Python and `deno` for JavaScript/TypeScript.

## Workflow

### Quick execution (preferred)

For one-off tasks, use `run` — it manages the session lifecycle automatically:

```bash
$BINARY run 'print("hello")'
$BINARY run --lang js 'console.log("hello")'
$BINARY run --cmd 'ls -la'
```

### Session-based workflow

For multi-step analysis where variables and imports need to persist between calls:

```bash
SESSION=$($BINARY start)
$BINARY exec $SESSION "import os; os.environ['AWS_DEFAULT_REGION'] = os.environ['AWS_REGION'] = '$AWS_REGION'; import boto3; client = boto3.client('s3'); print(client.list_buckets())"
$BINARY exec $SESSION 'print([b["Name"] for b in client.list_buckets()["Buckets"]])'
$BINARY stop $SESSION
```

Use `--clear-context` to reset state within a session without stopping it:

```bash
$BINARY exec $SESSION --clear-context --lang ts 'console.log("fresh start")'
```

## JSON Output

Pass `--json` for structured output:

```bash
$BINARY run --json 'print(42)'
```

Output format:

```json
{
  "stdout": "42\n",
  "stderr": "",
  "exitCode": 0,
  "executionTime": 123.4,
  "isError": false
}
```

For `start --json`:

```json
{"sessionId": "abc-123"}
```

## Environment

| Variable | Required | Description |
|----------|----------|-------------|
| `CLAUDE_PLUGIN_ROOT` | Auto | Set by Claude Code. Absolute path to the plugin root directory. Used to locate the binary in `bin/`. |
| `CUSTOM_AGENTCORE_INTERPRETER_ID` | Yes | Code interpreter identifier |
| `CUSTOM_AGENTCORE_AWS_PROFILE` | No | AWS shared config profile |
| `CUSTOM_AGENTCORE_AWS_REGION` | No | AWS region override |

- The sandbox has Python with boto3, pandas, numpy, and matplotlib pre-installed
- The sandbox IAM role has access to AWS services (S3, DynamoDB, CloudWatch Logs, CloudTrail, etc.)
- Sessions timeout after 15 minutes by default (override with `--timeout`)

## Important

- Execute exactly what the user asks — don't add extra steps, analysis, or embellishments they didn't request
- If you're unsure about any detail (region, resource name, time range, language), ask before executing
- Always stop sessions when done to avoid unnecessary charges (or use `run` which handles this automatically)
- For multi-step analysis, reuse the same session ID across `exec` calls (variables and imports persist)
- Pass code as a single-quoted string argument
- When running AWS CLI commands inside the sandbox, set `AWS_PAGER=''` to avoid pager errors since `less` is not available:
  ```bash
  $BINARY run --cmd "AWS_PAGER='' aws s3 ls"
  ```

## Troubleshooting

If something goes wrong, check these common issues before retrying:

| Error | Likely cause | Fix |
|-------|-------------|-----|
| Binary not found / permission denied | Wrong platform binary or missing binary for this OS/arch | Re-run platform detection. Check that `$BINARY` path exists and is executable. Currently only `darwin-arm64` is shipped. |
| `CUSTOM_AGENTCORE_INTERPRETER_ID` not set | Environment variable missing | Ask the user to set `CUSTOM_AGENTCORE_INTERPRETER_ID` in their Claude Code environment or shell profile. |
| AWS credentials error / `ExpiredTokenException` | Missing or expired AWS credentials on the host | Ask the user to refresh credentials (`aws sso login`, `aws configure`, or set `CUSTOM_AGENTCORE_AWS_PROFILE`). |
| Session start fails / connection refused | AgentCore service unreachable or interpreter ID invalid | Verify the interpreter ID is correct and the user has access. Check the AWS region is correct. |
| Timeout exceeded | Code ran longer than the session timeout | Retry with a longer `--timeout` value, or break the work into smaller steps. |
| `NoRegionError` / region not found inside sandbox | Region not passed into the sandbox code | Ensure both `AWS_DEFAULT_REGION` and `AWS_REGION` are set inside the sandbox code (see the "Set AWS_REGION" section above). |

When reporting errors to the user, include the full stderr output so they can diagnose the issue.

## Examples

**Python — query CloudTrail logs:**

```bash
$BINARY run "import os; os.environ['AWS_DEFAULT_REGION'] = os.environ['AWS_REGION'] = '$AWS_REGION'; import boto3; client = boto3.client('logs'); print(client.describe_log_groups())"
```

**JavaScript — process data:**

```bash
$BINARY run --lang js 'const data = [1,2,3]; console.log(data.reduce((a,b) => a+b))'
```

**TypeScript — typed analysis:**

```bash
$BINARY run --lang ts 'const x: number = 42; console.log(x)'
```

**Shell — list S3 buckets:**

```bash
$BINARY run --cmd "export AWS_DEFAULT_REGION='$AWS_REGION' AWS_REGION='$AWS_REGION'; AWS_PAGER='' aws s3 ls"
```

**Session — multi-step DynamoDB analysis:**

```bash
SESSION=$($BINARY start --timeout 1800)
$BINARY exec $SESSION "import os; os.environ['AWS_DEFAULT_REGION'] = os.environ['AWS_REGION'] = '$AWS_REGION'; import boto3; ddb = boto3.resource('dynamodb'); table = ddb.Table('my-table')"
$BINARY exec $SESSION 'response = table.scan(Limit=10); print(response["Items"])'
$BINARY stop $SESSION
```
