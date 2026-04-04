---
name: sandbox-exec
description: Execute Python code in a secure AWS AgentCore Code Interpreter sandbox. Use for AWS troubleshooting, querying CloudTrail logs, or running code in isolation.
allowed-tools: Bash(agentcore-sandbox-*)
user-invocable: true
argument-hint: "<python code or description of what to run>"
---

You have access to `agentcore-sandbox`, a CLI tool that executes Python code
in an isolated AWS AgentCore Code Interpreter sandbox.

## First: Detect Platform

Before running any commands, detect the platform to select the correct binary.
Follow the instructions in `references/platform-detection.md`.

Store the resolved binary name (e.g., `agentcore-sandbox-darwin-arm64`) and use
it for all commands below.

## Workflow

1. **Start a session**: Run `$BINARY start` to get a session ID
2. **Execute code**: Run `$BINARY exec <session-id> '<python code>'`
3. **Stop the session**: Run `$BINARY stop <session-id>` when done

## Environment

- The sandbox has Python with boto3, pandas, numpy, and matplotlib pre-installed
- The sandbox IAM role has read access to CloudWatch Logs (`aws-controltower/CloudTrailLogs`)
- Sessions timeout after 15 minutes if not explicitly stopped
- Set `CUSTOM_AGENTCORE_INTERPRETER_ID` env var to use a custom Code Interpreter (defaults to `aws.codeinterpreter.v1`)
- Set `CUSTOM_AGENTCORE_AWS_PROFILE` env var to use a specific AWS profile (defaults to the standard AWS credential chain)
- Set `CUSTOM_AGENTCORE_AWS_REGION` env var to override the AWS region

## Important

- Always stop the session when done to avoid unnecessary charges
- If the user asks to query CloudTrail or CloudWatch logs, use the sandbox
- For multi-step analysis, reuse the same session ID across exec calls
  (variables and imports persist between calls within a session)
- Pass Python code as a single-quoted string argument to exec

## Example

```bash
SESSION=$($BINARY start)
$BINARY exec $SESSION 'import boto3; client = boto3.client("logs", region_name="eu-west-1"); print(client.describe_log_groups())'
$BINARY stop $SESSION
```
