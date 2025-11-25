# Claude Integration with goose

This guide covers using Anthropic's Claude models with goose, including setup, configuration, and best practices.

## Overview

Claude is a family of advanced language models from Anthropic that excels at complex reasoning, coding tasks, and following detailed instructions. goose supports Claude through two primary integration methods:

1. **Anthropic API** - Direct API access to Claude models
2. **Claude Code CLI** - Pass-through provider using your Claude Code subscription

## Why Use Claude with goose?

Claude models are particularly well-suited for goose because they:

- Excel at complex, multi-step reasoning and planning
- Have strong coding capabilities across multiple languages
- Support large context windows (up to 200K tokens)
- Provide reliable tool/function calling for agent workflows
- Maintain consistent instruction-following behavior
- Support vision capabilities for analyzing images and screenshots

## Available Claude Models

goose supports the latest Claude models through the Anthropic provider:

| Model | Context Window | Best For | Notes |
|-------|----------------|----------|-------|
| `claude-sonnet-4-0` | 200K tokens | **Default** - Balanced performance and speed | Recommended for most use cases |
| `claude-sonnet-4-20250514` | 200K tokens | Specific dated version | For reproducibility |
| `claude-opus-4-0` | 200K tokens | Most capable model | Best for complex reasoning tasks |
| `claude-opus-4-20250514` | 200K tokens | Specific dated version | For reproducibility |
| `claude-3-7-sonnet-latest` | 200K tokens | Fast default model | Used for quick tasks |
| `claude-3-7-sonnet-20250219` | 200K tokens | Specific dated version | For reproducibility |
| `claude-3-opus-latest` | 200K tokens | Previous generation | Highly capable |

For the latest model information, see [Anthropic's model documentation](https://docs.anthropic.com/en/docs/about-claude/models).

## Setup and Configuration

### Option 1: Anthropic API (Recommended)

#### Prerequisites

- An [Anthropic API account](https://www.anthropic.com/)
- An API key from the [Anthropic Console](https://console.anthropic.com/)

#### Configuration

**goose Desktop:**

1. Click the sidebar button in the top-left
2. Navigate to `Settings` → `Models` → `Configure providers`
3. Select `Anthropic` from the provider list
4. Enter your API key when prompted
5. Click `Submit`

**goose CLI:**

1. Run the configuration command:
   ```bash
   goose configure
   ```

2. Select `Configure Providers` and choose `Anthropic`

3. Enter your API key when prompted

**Environment Variables:**

You can also configure Claude using environment variables:

```bash
export ANTHROPIC_API_KEY="your-api-key-here"
export ANTHROPIC_HOST="https://api.anthropic.com"  # Optional: Custom API endpoint
```

For development and testing, you can override settings per session:

```bash
ANTHROPIC_API_KEY="test-key" goose session
```

### Option 2: Claude Code CLI Provider

The Claude Code CLI provider allows you to use your Claude Code subscription instead of paying per API token.

#### Prerequisites

- Active [Claude Code subscription](https://www.anthropic.com/claude-code)
- Claude CLI installed and authenticated
- goose CLI or Desktop

#### Setup

1. Install and authenticate the Claude CLI:
   ```bash
   # Follow Anthropic's installation instructions
   # Authenticate with your Claude Code subscription
   ```

2. Configure goose to use the `claude-code` provider:
   ```bash
   goose configure
   ```

3. Select `claude-code` from the provider list

#### Features

- **200K context window** - Full access to Claude's extended context
- **Subscription-based** - No per-token charges
- **Native integration** - Uses Anthropic's official CLI tool
- **Automatic updates** - Benefits from CLI updates

**Limitations:**

- Requires active internet connection
- Depends on Claude CLI availability
- May have different rate limits than API access

See the [CLI Providers guide](https://block.github.io/goose/docs/guides/cli-providers) for more details.

## Using Claude with goose

### Basic Session

Start a goose session using Claude:

```bash
goose session
```

If Claude is your configured provider, goose will automatically use it. To specify Claude for a single session:

```bash
GOOSE_PROVIDER=anthropic goose session
```

### Multi-Model Configuration

You can configure Claude as your lead model with a faster/cheaper model for planning:

**goose Desktop:**

1. Navigate to `Settings` → `Models` → `Multi-model`
2. Enable multi-model mode
3. Set Claude as your lead model
4. Choose a fast model for planning (e.g., `claude-3-7-sonnet-latest`)

**goose CLI:**

Configure in your goose config file or use environment variables:

```bash
export GOOSE_MODEL="claude-sonnet-4-0"
export GOOSE_FAST_MODEL="claude-3-7-sonnet-latest"
```

See the [multi-model guide](https://block.github.io/goose/docs/guides/multi-model/creating-plans) for more information.

### Working with Images

Claude models support vision capabilities. You can ask goose to analyze screenshots, diagrams, or images:

```bash
# In a goose session
> Can you analyze this screenshot at /path/to/screenshot.png?
> What does this architecture diagram show? [attach image]
```

### Model Selection

To change your Claude model:

**goose Desktop:**
1. Click your current model name at the bottom of the app
2. Select `Change Model`
3. Choose your preferred Claude model

**goose CLI:**
```bash
# Set via environment variable
export GOOSE_MODEL="claude-opus-4-0"
goose session
```

## Best Practices

### 1. Choose the Right Model

- **claude-sonnet-4-0**: Best default choice for most tasks
- **claude-opus-4-0**: Use for highly complex reasoning, architecture decisions, or critical code
- **claude-3-7-sonnet-latest**: Use as fast model in multi-model setups

### 2. Leverage Context Windows

Claude's 200K token context window allows you to:
- Work with large codebases without frequent context resets
- Include comprehensive documentation
- Maintain longer conversation histories
- Process multiple files simultaneously

### 3. Use Clear Instructions

Claude excels with clear, detailed instructions. For best results:
- Break complex tasks into specific steps
- Provide examples of expected output
- Specify constraints and requirements upfront
- Use goose's [recipe system](https://block.github.io/goose/docs/guides/recipes/session-recipes) for repeated workflows

### 4. Take Advantage of Tool Calling

Claude's strong function-calling capabilities make it ideal for goose's agent workflows:
- File operations (read, write, edit)
- Shell command execution
- API interactions via MCP servers
- Multi-step task execution

### 5. Cost Optimization

If using the Anthropic API:
- Use `claude-sonnet-4-0` as your default (balanced cost/performance)
- Configure `claude-3-7-sonnet-latest` as your fast model for planning
- Enable multi-model mode to use cheaper models for simpler tasks
- Consider Claude Code CLI subscription for high-volume usage

### 6. Handle Rate Limits

Anthropic implements rate limits on API requests:
- goose automatically retries failed requests with exponential backoff
- For high-volume work, consider multiple API keys or Claude Code subscription
- Monitor your usage in the [Anthropic Console](https://console.anthropic.com/)

See the [handling rate limits guide](https://block.github.io/goose/docs/guides/handling-llm-rate-limits-with-goose) for more details.

## Advanced Configuration

### Custom API Endpoint

To use a custom Anthropic-compatible endpoint:

```bash
export ANTHROPIC_HOST="https://custom-endpoint.example.com"
```

### Provider Configuration File

For advanced setups, you can create custom provider configurations. See the [config files guide](https://block.github.io/goose/docs/guides/config-files) for details.

### AWS Bedrock Integration

To use Claude through AWS Bedrock instead of the direct API:

1. Configure AWS credentials in advance
2. Select `Amazon Bedrock` as your provider
3. Choose a Claude model (e.g., `anthropic.claude-sonnet-4-0-v1`)

Required environment variables:
```bash
export AWS_PROFILE="your-profile"
# OR
export AWS_ACCESS_KEY_ID="your-key"
export AWS_SECRET_ACCESS_KEY="your-secret"
export AWS_REGION="us-east-1"
```

See the [providers documentation](https://block.github.io/goose/docs/getting-started/providers) for more information.

### GCP Vertex AI Integration

To use Claude through Google Cloud Vertex AI:

1. Configure GCP credentials in advance
2. Select `GCP Vertex AI` as your provider
3. Choose a Claude model

Required environment variables:
```bash
export GCP_PROJECT_ID="your-project"
export GCP_LOCATION="us-central1"
```

### Multi-Provider Setups

You can configure multiple providers and switch between them:

```bash
# Use Claude for most work
export GOOSE_PROVIDER=anthropic

# Switch to a different provider for specific tasks
GOOSE_PROVIDER=openai goose session

# Use different models for lead vs. fast
export GOOSE_MODEL="claude-sonnet-4-0"
export GOOSE_FAST_MODEL="gpt-4o-mini"  # Cross-provider multi-model
```

## Troubleshooting

### API Key Issues

**Error: Missing ANTHROPIC_API_KEY**

Solution:
```bash
# Verify your API key is set
echo $ANTHROPIC_API_KEY

# Reconfigure if needed
goose configure
```

### Rate Limiting

**Error: 429 Too Many Requests**

Solutions:
- Wait for rate limit to reset (usually 1 minute)
- Reduce concurrent requests
- Upgrade your Anthropic plan
- Consider using Claude Code CLI subscription

### Context Length Errors

**Error: Prompt is too long**

Solutions:
- Use goose's [smart context management](https://block.github.io/goose/docs/guides/sessions/smart-context-management)
- Start a new session to reset context
- Reduce the number of files in context
- Use session summaries for long conversations

### Model Not Found

**Error: Model not available**

Solutions:
- Verify model name spelling
- Check [Anthropic's models page](https://docs.anthropic.com/en/docs/about-claude/models) for latest models
- Update your API key if using older models no longer available
- Some models require higher API tier access

## Additional Resources

- [Anthropic Documentation](https://docs.anthropic.com/)
- [Claude Model Documentation](https://docs.anthropic.com/en/docs/about-claude/models)
- [Anthropic Console](https://console.anthropic.com/)
- [goose Provider Documentation](https://block.github.io/goose/docs/getting-started/providers)
- [goose CLI Providers Guide](https://block.github.io/goose/docs/guides/cli-providers)
- [Multi-Model Configuration](https://block.github.io/goose/docs/guides/multi-model/creating-plans)
- [Responsible AI-Assisted Coding Guide](https://github.com/block/goose/blob/main/HOWTOAI.md)

## Contributing

Found an issue with Claude integration or have suggestions for improvements? We welcome contributions!

- [Open an issue](https://github.com/block/goose/issues)
- [Join our Discord](https://discord.gg/goose-oss)
- [Read the Contributing Guide](https://github.com/block/goose/blob/main/CONTRIBUTING.md)

## Support

Need help with Claude and goose?

- [Diagnostics & Reporting](https://block.github.io/goose/docs/troubleshooting/diagnostics-and-reporting)
- [Known Issues](https://block.github.io/goose/docs/troubleshooting/known-issues)
- [Discord Community](https://discord.gg/goose-oss)
- [GitHub Discussions](https://github.com/block/goose/discussions)
