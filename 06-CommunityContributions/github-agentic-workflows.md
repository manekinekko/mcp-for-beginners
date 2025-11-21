# GitHub Agentic Workflows for MCP Projects

## Overview

GitHub Agentic Workflows is an experimental framework that enables AI-powered automation directly within GitHub repositories. This guide shows you how to leverage agentic workflows in MCP projects for tasks like documentation maintenance, code review, and automated testing.

## What Are Agentic Workflows?

Agentic workflows combine:
- **GitHub Actions** infrastructure for reliable execution
- **AI coding agents** (GitHub Copilot, Claude, or custom engines) for intelligent automation
- **Natural language instructions** instead of complex YAML configurations
- **Safe output processing** for secure GitHub API interactions

Think of them as GitHub Actions that can "think" and make decisions based on repository context.

## Key Features

### 1. Markdown-Based Workflow Definition

Instead of complex YAML, define workflows using natural language:

```markdown
---
on:
  issues:
    types: [opened]
permissions:
  contents: read
safe-outputs:
  add-comment:
---

# Issue Triage Bot

Analyze issue #${{ github.event.issue.number }} and:
1. Categorize the issue type
2. Suggest appropriate labels
3. Post a helpful triage comment
```

### 2. Multiple AI Engine Support

Choose the right AI engine for your task:
- **Copilot** (default): Optimized for GitHub integration
- **Claude**: Advanced reasoning capabilities
- **Codex**: Legacy support
- **Custom**: Bring your own AI engine

### 3. Safe Output Processing

Separate AI processing from GitHub API writes:
- `create-issue`: Create issues from AI analysis
- `create-pull-request`: Generate PRs with code changes
- `add-comment`: Post comments to issues/PRs
- `create-discussion`: Start repository discussions

### 4. GitHub Context Integration

Access repository data with context expressions:
- `${{ github.event.issue.number }}`: Issue or PR number
- `${{ github.repository }}`: Repository name
- `${{ needs.activation.outputs.text }}`: Sanitized issue/PR content
- `${{ github.actor }}`: User who triggered the workflow

## Quick Start

### Prerequisites

Install the GitHub Agentic Workflows CLI extension:

```bash
gh extension install githubnext/gh-aw
```

### Create Your First Workflow

1. **Create workflow file** at `.github/workflows/doc-checker.md`:

```markdown
---
on:
  pull_request:
    types: [opened, edited]
permissions:
  contents: read
safe-outputs:
  add-comment:
---

# Documentation Checker

Review PR #${{ github.event.pull_request.number }} for documentation:

1. Check if code changes include corresponding documentation updates
2. Verify README accuracy
3. Suggest documentation improvements

Post a comment with your findings.
```

2. **Compile workflow**:

```bash
gh aw compile doc-checker
```

This generates `.github/workflows/doc-checker.lock.yml` that GitHub Actions can execute.

3. **Commit and push**:

```bash
git add .github/workflows/doc-checker.md .github/workflows/doc-checker.lock.yml
git commit -m "Add documentation checker workflow"
git push
```

## Real-World Examples for MCP Projects

### Example 1: MCP Server Documentation Sync

Keep documentation synchronized with code changes:

```markdown
---
on:
  push:
    branches: [main]
permissions:
  contents: read
safe-outputs:
  create-pull-request:
    draft: true
tools:
  github:
    toolsets: [default]
  edit:
---

# MCP Server Documentation Sync

Analyze recent code changes in MCP server implementations:

1. Review changes to server.py, index.ts, or Program.cs
2. Check if corresponding README.md files are up-to-date
3. Update documentation to reflect API changes
4. Create a draft PR with documentation improvements

Focus on:
- Tool definitions and parameters
- Resource schemas
- Configuration options
- Example usage
```

### Example 2: MCP Protocol Compliance Checker

Ensure MCP implementations follow specification:

```markdown
---
on:
  pull_request:
    types: [opened]
permissions:
  contents: read
safe-outputs:
  add-comment:
tools:
  github:
    toolsets: [repos, pull_requests]
  web-fetch:
---

# MCP Protocol Compliance Checker

Review PR #${{ github.event.pull_request.number }} for MCP specification compliance:

1. Fetch latest MCP spec from https://spec.modelcontextprotocol.io/
2. Check implementation against required features
3. Verify JSON-RPC 2.0 message format
4. Validate primitive implementations (tools, resources, prompts)
5. Check for proper error handling

Post a comment with compliance findings and suggestions.
```

### Example 3: Code Example Synchronization

Keep multi-language examples consistent:

```markdown
---
on:
  schedule:
    - cron: "0 9 * * 1"  # Weekly on Monday
permissions:
  contents: read
safe-outputs:
  create-issue:
    title-prefix: "[sync-check] "
    labels: [documentation, automated]
tools:
  github:
    toolsets: [repos, issues]
---

# Multi-Language Example Sync Checker

Check consistency across MCP code examples:

1. Compare implementations in:
   - 03-GettingStarted/01-first-server/solution/typescript/
   - 03-GettingStarted/01-first-server/solution/python/
   - 03-GettingStarted/01-first-server/solution/java/
   - 03-GettingStarted/01-first-server/solution/dotnet/

2. Verify all examples implement the same functionality
3. Check for consistency in tool definitions
4. Identify missing implementations

Create an issue listing any inconsistencies found.
```

## Best Practices for MCP Projects

### 1. Security First

Always use minimal permissions and safe-outputs:

```yaml
permissions:
  contents: read      # Minimal read access
  actions: read       # For workflow context

safe-outputs:
  create-pull-request:  # Secure PR creation
    draft: true         # Review before merging
```

### 2. Clear Instructions

Be specific about what the AI should do:

```markdown
# Good: Specific and actionable
Review the MCP server implementation in server.py:
1. Check tool registration follows MCP SDK patterns
2. Verify error handling includes proper JSON-RPC error codes
3. Validate resource URIs use consistent naming

# Avoid: Vague instructions
Check if the code is good.
```

### 3. Use Sanitized Context

Prefer `needs.activation.outputs.text` for security:

```markdown
# Recommended: Sanitized content
Analyze this issue content: "${{ needs.activation.outputs.text }}"

# Less secure: Raw event data
Title: ${{ github.event.issue.title }}
Body: ${{ github.event.issue.body }}
```

### 4. Set Appropriate Timeouts

Prevent runaway costs with time limits:

```yaml
timeout-minutes: 15      # Reasonable for most tasks
stop-after: +1mo         # Auto-disable after 1 month
```

### 5. Network Access Control

Limit network access to trusted domains:

```yaml
network:
  allowed:
    - defaults           # Basic infrastructure
    - python             # PyPI for Python projects
    - node               # NPM for Node.js projects
```

## Monitoring and Debugging

### View Workflow Logs

```bash
# Download logs for all agentic workflows
gh aw logs

# Download logs for specific workflow
gh aw logs doc-checker

# Filter by date range
gh aw logs --start-date -1w  # Last week's runs
```

### Inspect MCP Server Configurations

```bash
# List workflows with MCP configurations
gh aw mcp inspect

# Inspect specific workflow's MCP setup
gh aw mcp inspect doc-checker
```

### Validate Workflow Files

```bash
# Compile without generating .lock.yml (validation only)
gh aw compile doc-checker --no-emit

# Run security scanners
gh aw compile --strict --actionlint --zizmor
```

## Advanced Features

### Tool Configuration

Enable specific GitHub API capabilities:

```yaml
tools:
  github:
    toolsets: [repos, issues, pull_requests]  # Specific toolsets
  web-fetch:    # Fetch web content
  web-search:   # Search capabilities
  edit:         # File editing (required for code changes)
```

### Custom MCP Servers

Integrate custom MCP servers for specialized functionality:

```yaml
tools:
  mcp-servers:
    my-custom-server:
      command: "node"
      args: ["path/to/mcp-server.js"]
      allowed:
        - custom_function_1
        - custom_function_2
```

### Workflow Imports

Reuse common configurations:

```yaml
imports:
  - shared/security-notice.md
  - shared/mcp/tools-setup.md
```

## Troubleshooting

### Common Issues

**Issue**: Workflow not triggering
- Check `.lock.yml` file is committed
- Verify trigger configuration matches events
- Review permissions settings

**Issue**: AI making incorrect changes
- Add more specific instructions
- Provide examples in workflow markdown
- Use safe-outputs to review changes before merging

**Issue**: Permission errors
- Ensure minimal required permissions are granted
- Use safe-outputs for write operations
- Check GitHub token scopes

### Getting Help

- **Documentation**: `.github/instructions/github-agentic-workflows.instructions.md`
- **Examples**: `.github/agents/` directory
- **Community**: [GitHub Next Discussions](https://github.com/githubnext/discussions)

## Resources

### Official Documentation
- [GitHub Agentic Workflows Documentation](https://githubnext.github.io/gh-aw/)
- [MCP Specification](https://spec.modelcontextprotocol.io/)
- [GitHub Actions Documentation](https://docs.github.com/actions)

### Example Workflows
- Update Docs: `.github/workflows/update-docs.md`
- Agent Templates: `.github/agents/`

### Tools and Extensions
- [gh-aw CLI Extension](https://github.com/githubnext/gh-aw)
- [GitHub CLI](https://cli.github.com/)

## Contributing

When creating agentic workflows for MCP projects:

1. **Test thoroughly** with `--no-emit` flag first
2. **Document purpose** clearly in workflow description
3. **Use draft PRs** for changes requiring review
4. **Set time limits** to prevent excessive AI usage
5. **Follow security best practices** from this guide

## Conclusion

GitHub Agentic Workflows provide powerful automation capabilities for MCP projects. By combining AI intelligence with GitHub's robust infrastructure, you can automate documentation maintenance, code review, and quality assurance tasks while maintaining security and control.

Start with simple workflows like documentation checkers, then gradually expand to more complex automation as you become comfortable with the framework.
