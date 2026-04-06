# my2centsai-blog-samples

A Claude Code plugin marketplace containing plugins for AWS development and troubleshooting.

## Repo Structure

```
.claude-plugin/
  marketplace.json          # Marketplace manifest — lists all plugins
plugins/
  <plugin-name>/
    .claude-plugin/
      plugin.json           # Plugin metadata (name, version, author, keywords)
    skills/
      <skill-name>/
        SKILL.md            # Skill definition (YAML frontmatter + markdown instructions)
        references/         # Optional supporting docs referenced by the skill
    bin/                    # Platform-specific binaries: <name>-<os>-<arch>[.exe]
    LICENSE
```

## Conventions

- **Plugin names**: kebab-case (e.g., `aws-bedrock-agentcore-codeinterpreter`)
- **Skill names**: kebab-case (e.g., `code-interpreter-execute`)
- **Binary naming**: `<tool>-<os>-<arch>` where os is `darwin|linux|windows` and arch is `arm64|amd64`
- **SKILL.md frontmatter**: must include `name`, `description`, `allowed-tools`, `user-invocable`
- **plugin.json**: must include `name`, `version`, `description`, `author`, `license`, `keywords`
- **marketplace.json**: plugin entries must have `name`, `version`, `source`, `category`
- Plugin `name` in marketplace.json must match `name` in the corresponding plugin.json

## Adding a New Plugin

1. Create `plugins/<plugin-name>/.claude-plugin/plugin.json` with required metadata
2. Create `plugins/<plugin-name>/skills/<skill-name>/SKILL.md` with frontmatter and instructions
3. Add the plugin entry to `.claude-plugin/marketplace.json` under `plugins`
4. Add a LICENSE file to the plugin directory
5. Place any platform binaries in `plugins/<plugin-name>/bin/`
