# my2centsai-blog-samples

A Claude Code plugin marketplace for AWS development and troubleshooting tools.

## Plugins

### aws-bedrock-agentcore-codeinterpreter

Execute Python, JavaScript, TypeScript, and shell commands in a secure [AWS Bedrock AgentCore](https://aws.amazon.com/bedrock/agentcore/) Code Interpreter sandbox. Designed for AWS troubleshooting, log analysis, and isolated code execution.

**Supported languages:** Python (with boto3, pandas, numpy, matplotlib), JavaScript, TypeScript, Shell

## Prerequisites

- [Claude Code](https://claude.ai/code) installed
- AWS credentials configured on your machine (SSO, env vars, or shared config)
- An AgentCore Code Interpreter identifier (set as `CUSTOM_AGENTCORE_INTERPRETER_ID`)

## Installation

Install the plugin from this repository:

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

The skill supports two workflows:

- **Quick execution** (`run`): One-shot commands that automatically manage the sandbox session lifecycle
- **Session-based** (`start` / `exec` / `stop`): Multi-step analysis where variables and imports persist between calls

## Architecture

```
marketplace.json          Indexes available plugins
  |
  +-- plugin.json         Plugin metadata and identity
        |
        +-- SKILL.md      Skill instructions (what Claude executes)
        |
        +-- bin/           Platform-specific CLI binaries
              |
              +-- agentcore-sandbox-<os>-<arch>
```

The skill wraps the `agentcore-sandbox` CLI binary, which communicates with the AWS AgentCore Code Interpreter service. The binary handles session management, code execution, and output streaming.

## Platform Support

The `agentcore-sandbox` binary is currently available for:

- macOS ARM64 (Apple Silicon)

Additional platforms (macOS AMD64, Linux, Windows) will be added in future releases.

## Contributing

1. Fork this repository
2. Create a new plugin under `plugins/<your-plugin-name>/`
3. Follow the structure documented in [CLAUDE.md](CLAUDE.md)
4. Submit a pull request

## License

This project is licensed under the Apache License 2.0 - see [LICENSE](LICENSE) for details.
