# aws-bedrock-agentcore-codeinterpreter

A [Claude Code](https://claude.ai/code) plugin that executes Python, JavaScript, TypeScript, and shell commands in a secure [AWS Bedrock AgentCore](https://aws.amazon.com/bedrock/agentcore/) Code Interpreter sandbox. Designed for AWS troubleshooting, log analysis, and isolated code execution.

## Supported Languages

- **Python** (with boto3, pandas, numpy, matplotlib pre-installed)
- **JavaScript**
- **TypeScript**
- **Shell** (AWS CLI available)

## Prerequisites

- [Claude Code](https://claude.ai/code) installed
- AWS credentials configured on your machine (SSO, env vars, or shared config)
- An AgentCore Code Interpreter identifier (set as `CUSTOM_AGENTCORE_INTERPRETER_ID`)

## Installation

```bash
claude plugin add fabiodouek/my2centsai-blog-samples
```

## Configuration

Set the required environment variables before using the plugin:

| Variable | Required | Description |
|----------|----------|-------------|
| `CUSTOM_AGENTCORE_INTERPRETER_ID` | Yes | Your AgentCore Code Interpreter identifier |
| `CUSTOM_AGENTCORE_AWS_PROFILE` | No | AWS shared config profile to use |
| `CUSTOM_AGENTCORE_AWS_REGION` | No | AWS region override (defaults to `AWS_REGION`) |

You can set these in your shell profile, `.env` file, or Claude Code settings.

## Usage

Invoke the skill from Claude Code:

```
/code-interpreter-execute list my S3 buckets
```

```
/code-interpreter-execute import boto3; print(boto3.client('s3').list_buckets())
```

### Workflows

**Quick execution** (preferred) — one-shot commands that automatically manage the sandbox session lifecycle:

```bash
agentcore-sandbox run 'print("hello")'
agentcore-sandbox run --lang js 'console.log("hello")'
agentcore-sandbox run --cmd 'ls -la'
```

**Session-based** — multi-step analysis where variables and imports persist between calls:

```bash
SESSION=$(agentcore-sandbox start)
agentcore-sandbox exec $SESSION "<code>"
agentcore-sandbox exec $SESSION "<more code>"
agentcore-sandbox stop $SESSION
```

### Commands

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

## Architecture

```
plugins/aws-bedrock-agentcore-codeinterpreter/
  .claude-plugin/
    plugin.json           # Plugin metadata and identity
  skills/
    code-interpreter-execute/
      SKILL.md            # Skill instructions (what Claude executes)
      references/         # Supporting docs for platform detection
  bin/
    agentcore-sandbox-*   # Platform-specific CLI binaries
  LICENSE
```

The skill wraps the `agentcore-sandbox` CLI binary, which communicates with the AWS AgentCore Code Interpreter service. The binary handles session management, code execution, and output streaming.

## Platform Support

| Platform | Architecture | Status |
|----------|-------------|--------|
| macOS | ARM64 (Apple Silicon) | Available |
| macOS | AMD64 | Planned |
| Linux | ARM64 / AMD64 | Planned |
| Windows | AMD64 | Planned |

## Examples

**Python — query CloudTrail logs:**

```bash
agentcore-sandbox run "import boto3; client = boto3.client('logs'); print(client.describe_log_groups())"
```

**JavaScript — process data:**

```bash
agentcore-sandbox run --lang js 'const data = [1,2,3]; console.log(data.reduce((a,b) => a+b))'
```

**Shell — list S3 buckets:**

```bash
agentcore-sandbox run --cmd "AWS_PAGER='' aws s3 ls"
```

**Session — multi-step DynamoDB analysis:**

```bash
SESSION=$(agentcore-sandbox start --timeout 1800)
agentcore-sandbox exec $SESSION "import boto3; ddb = boto3.resource('dynamodb'); table = ddb.Table('my-table')"
agentcore-sandbox exec $SESSION 'response = table.scan(Limit=10); print(response["Items"])'
agentcore-sandbox stop $SESSION
```

## Troubleshooting

| Error | Likely Cause | Fix |
|-------|-------------|-----|
| Binary not found / permission denied | Wrong platform or missing binary | Check platform support above. Currently only `darwin-arm64` is shipped. |
| `CUSTOM_AGENTCORE_INTERPRETER_ID` not set | Env var missing | Set `CUSTOM_AGENTCORE_INTERPRETER_ID` in your shell profile or Claude Code settings. |
| AWS credentials error | Missing or expired credentials | Run `aws sso login` or set `CUSTOM_AGENTCORE_AWS_PROFILE`. |
| Session start fails | Service unreachable or invalid interpreter ID | Verify the interpreter ID and AWS region. |
| Timeout exceeded | Code ran too long | Use a longer `--timeout` or break work into smaller steps. |

## License

Apache License 2.0 — see [LICENSE](LICENSE).
