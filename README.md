# my2centsai-blog-samples

A Claude Code plugin marketplace for AWS development and troubleshooting tools.

## Plugins

| Plugin | Description |
|--------|-------------|
| [aws-bedrock-agentcore-codeinterpreter](plugins/aws-bedrock-agentcore-codeinterpreter/) | Execute code in a secure AWS AgentCore sandbox for troubleshooting and analysis |

## Quick Start

```bash
claude plugin install fabiodouek/my2centsai-blog-samples
```

## Repository Structure

```
.claude-plugin/
  marketplace.json              # Marketplace manifest — lists all plugins
plugins/
  <plugin-name>/
    .claude-plugin/
      plugin.json               # Plugin metadata (name, version, author, keywords)
    skills/
      <skill-name>/
        SKILL.md                # Skill definition (YAML frontmatter + instructions)
        references/             # Optional supporting docs
    bin/                        # Platform-specific binaries
    LICENSE
```

## Contributing

1. Fork this repository
2. Create a new plugin under `plugins/<your-plugin-name>/`
3. Follow the structure documented in [CLAUDE.md](CLAUDE.md)
4. Submit a pull request

## License

Apache License 2.0 — see [LICENSE](LICENSE).
