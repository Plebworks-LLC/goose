# goose Crate Architecture

This document provides a comprehensive overview of the `goose` crate architecture, detailing its core components, data flows, and interactions.

## Table of Contents

- [Overview](#overview)
- [High-Level Architecture](#high-level-architecture)
- [Core Components](#core-components)
- [Data Flow](#data-flow)
- [Module Dependencies](#module-dependencies)
- [Component Details](#component-details)

## Overview

The `goose` crate is the core library powering the goose AI agent. It implements a sophisticated agent system that orchestrates LLM interactions, tool execution, session management, and extensibility through MCP (Model Context Protocol) servers.

**Key Statistics:**
- **163 Rust source files**
- **27 main modules**
- **Multiple provider integrations** (Anthropic, OpenAI, Google, AWS, Azure, etc.)
- **Extensible architecture** via MCP protocol

## High-Level Architecture

```mermaid
graph TB
    subgraph "Presentation Layer"
        CLI[CLI Interface]
        Desktop[Desktop App]
        Server[goose Server]
    end

    subgraph "Agent Core - goose crate"
        Agent[Agent Orchestrator]
        Session[Session Manager]
        Execution[Execution Manager]
    end

    subgraph "Provider Layer"
        Provider[Provider Interface]
        Anthropic[Anthropic]
        OpenAI[OpenAI]
        Others[Other Providers]
    end

    subgraph "Extension System"
        ExtMgr[Extension Manager]
        MCP[MCP Clients]
        Tools[Tool System]
    end

    subgraph "Support Services"
        Config[Configuration]
        Permission[Permission System]
        Context[Context Management]
        Security[Security Inspector]
    end

    CLI --> Agent
    Desktop --> Agent
    Server --> Agent

    Agent --> Session
    Agent --> Execution
    Agent --> Provider
    Agent --> ExtMgr

    Provider --> Anthropic
    Provider --> OpenAI
    Provider --> Others

    ExtMgr --> MCP
    ExtMgr --> Tools

    Agent --> Config
    Agent --> Permission
    Agent --> Context
    Agent --> Security

    style Agent fill:#4A90E2,color:#fff
    style Provider fill:#50C878,color:#fff
    style ExtMgr fill:#F5A623,color:#fff
```

## Core Components

### 1. Agent System (`agents/`)

The Agent module is the heart of goose, orchestrating all agent operations.

```mermaid
graph TB
    subgraph "Agent Core"
        Agent[Agent]
        PromptMgr[Prompt Manager]
        ExtMgr[Extension Manager]
        ToolRoute[Tool Route Manager]
    end

    subgraph "Agent Sub-Components"
        Retry[Retry Manager]
        SubRecipe[Sub-Recipe Manager]
        SubAgent[Subagent Handler]
        Tasks[Tasks Manager]
    end

    subgraph "Tool System"
        Platform[Platform Tools]
        Recipe[Recipe Tools]
        Router[Router Tools]
        Schedule[Schedule Tool]
        Final[Final Output Tool]
    end

    subgraph "Extensions"
        ChatRecall[ChatRecall Extension]
        Todo[Todo Extension]
        Skills[Skills Extension]
        ExtMgrExt[Extension Manager Extension]
    end

    Agent --> PromptMgr
    Agent --> ExtMgr
    Agent --> ToolRoute
    Agent --> Retry
    Agent --> SubRecipe
    Agent --> SubAgent
    Agent --> Tasks

    Agent --> Platform
    Agent --> Recipe
    Agent --> Router
    Agent --> Schedule
    Agent --> Final

    ExtMgr --> ChatRecall
    ExtMgr --> Todo
    ExtMgr --> Skills
    ExtMgr --> ExtMgrExt

    style Agent fill:#4A90E2,color:#fff
    style ExtMgr fill:#F5A623,color:#fff
```

**Key Responsibilities:**
- **Agent (`agent.rs`)**: Main orchestrator for agent loop, message processing, and tool execution
- **Prompt Manager**: Builds and manages system prompts with dynamic components
- **Extension Manager**: Manages MCP clients and external tool integrations
- **Retry Manager**: Handles retry logic for failed operations
- **Subagent Handler**: Manages subagent spawning and coordination
- **Tool Execution**: Dispatches and monitors tool calls

### 2. Provider System (`providers/`)

Abstraction layer for LLM providers with support for 20+ providers.

```mermaid
graph LR
    subgraph "Provider Interface"
        Base[Provider Trait]
        Factory[Provider Factory]
        Registry[Provider Registry]
    end

    subgraph "Direct API Providers"
        Anthropic[Anthropic]
        OpenAI[OpenAI]
        Google[Google Gemini]
        Groq[Groq]
        Mistral[Mistral AI]
        XAI[xAI]
    end

    subgraph "Cloud Providers"
        Bedrock[AWS Bedrock]
        Azure[Azure OpenAI]
        GCP[GCP Vertex AI]
        Databricks[Databricks]
        Sagemaker[SageMaker TGI]
        Snowflake[Snowflake]
    end

    subgraph "CLI Providers"
        ClaudeCode[Claude Code CLI]
        CursorAgent[Cursor Agent]
        GeminiCLI[Gemini CLI]
    end

    subgraph "Aggregators"
        OpenRouter[OpenRouter]
        Tetrate[Tetrate Router]
        LiteLLM[LiteLLM]
    end

    subgraph "Local Providers"
        Ollama[Ollama]
        Venice[Venice AI]
    end

    Base --> Factory
    Factory --> Anthropic
    Factory --> OpenAI
    Factory --> Google
    Factory --> Groq
    Factory --> Bedrock
    Factory --> Azure
    Factory --> GCP
    Factory --> ClaudeCode
    Factory --> OpenRouter
    Factory --> Ollama

    Registry --> Factory

    style Base fill:#50C878,color:#fff
    style Factory fill:#4A90E2,color:#fff
```

**Key Features:**
- **Unified Interface**: `Provider` trait abstracts all LLM interactions
- **Streaming Support**: Real-time response streaming
- **Multi-Model**: Lead/worker model pattern for efficiency
- **Format Adapters**: Convert between provider-specific formats
- **Retry Logic**: Automatic retry with exponential backoff
- **Rate Limiting**: Built-in rate limit handling

### 3. Conversation Management (`conversation/`)

Manages message history and conversation state.

```mermaid
graph TB
    subgraph "Conversation System"
        Conv[Conversation]
        Msg[Message]
        Content[Message Content]
    end

    subgraph "Message Types"
        Text[Text Content]
        ToolReq[Tool Request]
        ToolResp[Tool Response]
        Image[Image Content]
        Thinking[Thinking Content]
    end

    subgraph "Message Metadata"
        Meta[Message Metadata]
        Visibility[Visibility Flags]
        Tokens[Token Counts]
    end

    Conv --> Msg
    Msg --> Content
    Msg --> Meta

    Content --> Text
    Content --> ToolReq
    Content --> ToolResp
    Content --> Image
    Content --> Thinking

    Meta --> Visibility
    Meta --> Tokens

    style Conv fill:#9B59B6,color:#fff
```

**Key Responsibilities:**
- Message validation and ordering
- Role management (User, Assistant, System)
- Tool call/response tracking
- Message merging and deduplication
- Serialization/deserialization

### 4. Session Management (`session/`)

Handles session lifecycle, persistence, and state.

```mermaid
graph TB
    subgraph "Session System"
        SessionMgr[Session Manager]
        Session[Session]
        ExtData[Extension Data]
    end

    subgraph "Session Types"
        Interactive[Interactive Session]
        Background[Background/Scheduled]
        SubTask[Sub-Task Session]
    end

    subgraph "Session State"
        History[Chat History]
        Context[Session Context]
        Insights[Session Insights]
    end

    subgraph "Extension State"
        EnabledExt[Enabled Extensions]
        TodoState[Todo State]
        CustomData[Custom Extension Data]
    end

    SessionMgr --> Session
    Session --> ExtData
    Session --> History
    Session --> Context
    Session --> Insights

    Session --> Interactive
    Session --> Background
    Session --> SubTask

    ExtData --> EnabledExt
    ExtData --> TodoState
    ExtData --> CustomData

    style SessionMgr fill:#E74C3C,color:#fff
    style Session fill:#3498DB,color:#fff
```

**Key Features:**
- Session creation and lifecycle management
- Conversation persistence and loading
- Extension state management
- Session diagnostics and insights
- Multi-session coordination

### 5. Extension System (`agents/extension*.rs`, `agents/mcp_client.rs`)

Provides extensibility through MCP (Model Context Protocol).

```mermaid
graph TB
    subgraph "Extension Manager"
        ExtMgr[Extension Manager]
        ExtConfig[Extension Config]
    end

    subgraph "MCP Client Layer"
        McpClient[MCP Client]
        McpTrait[MCP Client Trait]
    end

    subgraph "MCP Operations"
        ListTools[List Tools]
        CallTool[Call Tool]
        ListResources[List Resources]
        ReadResource[Read Resource]
        ListPrompts[List Prompts]
        GetPrompt[Get Prompt]
    end

    subgraph "Transport Layer"
        Stdio[Stdio Transport]
        SSE[SSE Transport]
        HTTP[HTTP Transport]
    end

    subgraph "Built-in Extensions"
        Platform[Platform Extensions]
        Malware[Malware Check]
        ChatRecall[Chat Recall]
        Todo[Todo Extension]
        Skills[Skills Extension]
    end

    ExtMgr --> ExtConfig
    ExtMgr --> McpClient
    McpClient --> McpTrait

    McpTrait --> ListTools
    McpTrait --> CallTool
    McpTrait --> ListResources
    McpTrait --> ReadResource
    McpTrait --> ListPrompts
    McpTrait --> GetPrompt

    McpClient --> Stdio
    McpClient --> SSE
    McpClient --> HTTP

    ExtMgr --> Platform
    ExtMgr --> Malware
    ExtMgr --> ChatRecall
    ExtMgr --> Todo
    ExtMgr --> Skills

    style ExtMgr fill:#F5A623,color:#fff
    style McpClient fill:#16A085,color:#fff
```

**Key Features:**
- Dynamic extension loading and management
- MCP protocol implementation
- Tool registration and routing
- Resource management
- OAuth flow support for authenticated extensions
- Notification handling and streaming

### 6. Recipe System (`recipe/`)

Workflow and task automation through recipes.

```mermaid
graph TB
    subgraph "Recipe System"
        Recipe[Recipe]
        Builder[Recipe Builder]
        Template[Template Recipe]
    end

    subgraph "Recipe Components"
        Instructions[Instructions]
        Prompt[Initial Prompt]
        Settings[Settings]
        Params[Parameters]
    end

    subgraph "Recipe Features"
        SubRecipes[Sub-Recipes]
        Extensions[Recipe Extensions]
        Response[Response Schema]
        Retry[Retry Config]
    end

    subgraph "Recipe Tools"
        Dynamic[Dynamic Task Tools]
        SubRecipeTool[Sub-Recipe Tools]
    end

    Recipe --> Builder
    Recipe --> Template

    Recipe --> Instructions
    Recipe --> Prompt
    Recipe --> Settings
    Recipe --> Params

    Recipe --> SubRecipes
    Recipe --> Extensions
    Recipe --> Response
    Recipe --> Retry

    Recipe --> Dynamic
    Recipe --> SubRecipeTool

    style Recipe fill:#8E44AD,color:#fff
```

**Key Features:**
- YAML/JSON recipe definitions
- Parameter templating
- Sub-recipe composition
- Extension activation
- Response schema validation
- Local and built-in recipe management

### 7. Configuration System (`config/`)

Global configuration and settings management.

```mermaid
graph TB
    subgraph "Configuration"
        Config[Config Singleton]
        Paths[Path Management]
        Experiments[Experiment Manager]
    end

    subgraph "Provider Config"
        ProviderReg[Provider Registry]
        Declarative[Declarative Providers]
        Signup[Signup Flows]
    end

    subgraph "Extension Config"
        ExtRegistry[Extension Registry]
        EnabledExt[Enabled Extensions]
    end

    subgraph "Permission Config"
        PermMgr[Permission Manager]
        ToolPerms[Tool Permissions]
    end

    subgraph "Settings"
        Mode[Goose Mode]
        Secrets[Secret Storage]
        Params[Parameters]
    end

    Config --> Paths
    Config --> Experiments
    Config --> ProviderReg
    Config --> ExtRegistry
    Config --> PermMgr
    Config --> Mode

    ProviderReg --> Declarative
    ProviderReg --> Signup

    ExtRegistry --> EnabledExt

    PermMgr --> ToolPerms

    Config --> Secrets
    Config --> Params

    style Config fill:#2C3E50,color:#fff
```

**Key Responsibilities:**
- Global singleton configuration
- Provider registration and credentials
- Extension management
- Permission settings
- Path resolution (config, data, state)
- Feature flag management

### 8. Context Management (`context_mgmt/`)

Manages conversation context and token limits.

```mermaid
graph LR
    subgraph "Context Management"
        Check[Context Check]
        Compact[Message Compaction]
        Counter[Token Counter]
    end

    subgraph "Compaction Strategy"
        Threshold[Threshold Check]
        Summarize[Summarization]
        Preserve[Message Preservation]
    end

    subgraph "Token Accounting"
        Count[Token Counting]
        Estimate[Usage Estimation]
        Pricing[Cost Calculation]
    end

    Check --> Threshold
    Threshold --> Compact
    Compact --> Summarize
    Compact --> Preserve

    Check --> Counter
    Counter --> Count
    Counter --> Estimate
    Counter --> Pricing

    style Compact fill:#1ABC9C,color:#fff
```

**Key Features:**
- Automatic context compaction at 80% threshold
- Smart message summarization using LLM
- Token counting for multiple providers
- Cost estimation
- Manual compaction support

### 9. Permission System (`permission/`)

Security and permission management for tool execution.

```mermaid
graph TB
    subgraph "Permission System"
        Inspector[Permission Inspector]
        Judge[Permission Judge]
        Store[Permission Store]
    end

    subgraph "Permission Types"
        Allow[Always Allow]
        AllowOnce[Allow Once]
        Deny[Always Deny]
        Ask[Always Ask]
    end

    subgraph "Tool Classification"
        ReadOnly[Read-Only Tools]
        Write[Write Tools]
        Dangerous[Dangerous Tools]
    end

    subgraph "Security Checks"
        PathCheck[Path Validation]
        ArgCheck[Argument Inspection]
        PatternMatch[Pattern Matching]
    end

    Inspector --> Judge
    Judge --> Store

    Store --> Allow
    Store --> AllowOnce
    Store --> Deny
    Store --> Ask

    Judge --> ReadOnly
    Judge --> Write
    Judge --> Dangerous

    Inspector --> PathCheck
    Inspector --> ArgCheck
    Inspector --> PatternMatch

    style Inspector fill:#E67E22,color:#fff
    style Judge fill:#C0392B,color:#fff
```

**Key Features:**
- Tool-level permission control
- Argument validation
- Automatic read-only tool detection
- User confirmation workflows
- Permission persistence

### 10. Security System (`security/`)

Advanced security inspection and threat detection.

```mermaid
graph TB
    subgraph "Security Inspector"
        Inspector[Security Inspector]
        Scanner[Malware Scanner]
    end

    subgraph "Detection"
        Prompt[Prompt Injection]
        Malware[Malware Detection]
        Patterns[Pattern Analysis]
    end

    subgraph "Response"
        Block[Block Execution]
        Warn[Warning]
        Log[Security Logging]
    end

    Inspector --> Scanner

    Inspector --> Prompt
    Inspector --> Malware
    Inspector --> Patterns

    Inspector --> Block
    Inspector --> Warn
    Inspector --> Log

    style Inspector fill:#C0392B,color:#fff
```

### 11. Execution System (`execution/`)

Session execution lifecycle management.

```mermaid
graph LR
    subgraph "Execution Manager"
        Manager[Execution Manager]
        Lifecycle[Session Lifecycle]
    end

    subgraph "Execution Modes"
        Interactive[Interactive]
        Background[Background]
        SubTask[Sub-Task]
    end

    subgraph "Agent Isolation"
        Instance[Agent Instance]
        Provider[Provider Instance]
        Extensions[Extension State]
    end

    Manager --> Lifecycle

    Lifecycle --> Interactive
    Lifecycle --> Background
    Lifecycle --> SubTask

    Manager --> Instance
    Manager --> Provider
    Manager --> Extensions

    style Manager fill:#27AE60,color:#fff
```

## Data Flow

### Agent Reply Loop

```mermaid
sequenceDiagram
    participant User
    participant Agent
    participant Provider
    participant Tools
    participant Extensions

    User->>Agent: Send Message
    Agent->>Agent: Build Context
    Agent->>Agent: Check Permissions
    Agent->>Agent: Compact if needed
    Agent->>Provider: Stream Completion
    Provider-->>Agent: Assistant Message

    alt Tool Calls Present
        Agent->>Agent: Inspect Tool Calls
        Agent->>Agent: Check Security

        alt Requires Approval
            Agent->>User: Request Confirmation
            User->>Agent: Approve/Deny
        end

        alt Approved
            Agent->>Tools: Execute Tool
            alt MCP Tool
                Tools->>Extensions: Call MCP Extension
                Extensions-->>Tools: Tool Result
            end
            Tools-->>Agent: Tool Response
            Agent->>Provider: Continue with Result
        end
    end

    Provider-->>Agent: Final Response
    Agent->>Agent: Save to Session
    Agent-->>User: Display Response
```

### Extension Tool Execution

```mermaid
sequenceDiagram
    participant Agent
    participant ExtMgr as Extension Manager
    participant MCP as MCP Client
    participant Server as MCP Server

    Agent->>ExtMgr: Call Tool
    ExtMgr->>ExtMgr: Find Extension
    ExtMgr->>MCP: Call Tool Request

    alt Stdio Transport
        MCP->>Server: JSON-RPC over stdio
    else SSE Transport
        MCP->>Server: SSE Request
    else HTTP Transport
        MCP->>Server: HTTP Request
    end

    Server->>Server: Process Tool

    alt Streaming Response
        loop Stream Chunks
            Server-->>MCP: Notification
            MCP-->>ExtMgr: Notification Event
            ExtMgr-->>Agent: Stream Update
        end
    end

    Server-->>MCP: Tool Result
    MCP-->>ExtMgr: Result
    ExtMgr-->>Agent: Tool Response
```

### Context Compaction Flow

```mermaid
sequenceDiagram
    participant Agent
    participant Context as Context Manager
    participant Provider
    participant Conversation

    Agent->>Context: Check Context Size
    Context->>Context: Count Tokens

    alt Exceeds Threshold
        Context->>Context: Identify Messages to Compact
        Context->>Context: Preserve Recent User Message
        Context->>Context: Preserve Tool Messages
        Context->>Provider: Summarize Messages
        Provider-->>Context: Summary
        Context->>Context: Create Summary Message
        Context->>Context: Mark Old Messages Hidden
        Context->>Conversation: Update Conversation
        Conversation-->>Agent: Compacted Conversation
    else Below Threshold
        Context-->>Agent: No Action Needed
    end
```

## Module Dependencies

```mermaid
graph TD
    subgraph "Core"
        Agent[agents]
        Conversation[conversation]
        Session[session]
    end

    subgraph "Providers"
        Provider[providers]
        Model[model]
    end

    subgraph "Infrastructure"
        Config[config]
        Execution[execution]
        Context[context_mgmt]
    end

    subgraph "Extensions"
        Recipe[recipe]
        MCP[mcp_utils]
        Commands[slash_commands]
    end

    subgraph "Support"
        Permission[permission]
        Security[security]
        Tracing[tracing]
        Utils[utils]
    end

    Agent --> Conversation
    Agent --> Session
    Agent --> Provider
    Agent --> Config
    Agent --> Permission
    Agent --> Security
    Agent --> Context
    Agent --> Recipe
    Agent --> MCP

    Session --> Conversation
    Session --> Config

    Provider --> Model
    Provider --> Config

    Execution --> Agent
    Execution --> Session
    Execution --> Provider

    Recipe --> Agent
    Recipe --> Config

    Context --> Conversation
    Context --> Provider

    Permission --> Config
    Security --> Config

    Tracing --> Config

    style Agent fill:#4A90E2,color:#fff
    style Provider fill:#50C878,color:#fff
    style Config fill:#2C3E50,color:#fff
```

## Component Details

### Tool Routing and Execution

```mermaid
graph TB
    subgraph "Tool Request Flow"
        Request[Tool Request]
        Router[Tool Router]
        RouteIndex[Route Index Manager]
    end

    subgraph "Tool Types"
        Platform[Platform Tools]
        MCP[MCP Tools]
        Recipe[Recipe Tools]
        Dynamic[Dynamic Tools]
        Subagent[Subagent Tools]
    end

    subgraph "Execution Pipeline"
        Inspector[Tool Inspector]
        Security[Security Check]
        Permission[Permission Check]
        Execute[Execute Tool]
    end

    subgraph "Result Handling"
        Stream[Stream Results]
        Notify[Notifications]
        Response[Format Response]
    end

    Request --> Router
    Router --> RouteIndex
    RouteIndex --> Platform
    RouteIndex --> MCP
    RouteIndex --> Recipe
    RouteIndex --> Dynamic
    RouteIndex --> Subagent

    Platform --> Inspector
    MCP --> Inspector
    Recipe --> Inspector
    Dynamic --> Inspector
    Subagent --> Inspector

    Inspector --> Security
    Security --> Permission
    Permission --> Execute

    Execute --> Stream
    Execute --> Notify
    Execute --> Response

    style Router fill:#9B59B6,color:#fff
    style Execute fill:#E67E22,color:#fff
```

### Subagent System

```mermaid
graph TB
    subgraph "Subagent Execution"
        Handler[Subagent Handler]
        TaskConfig[Task Config]
        Executor[Executor]
    end

    subgraph "Execution Modes"
        Plan[Plan Mode]
        Explore[Explore Mode]
        General[General Purpose]
    end

    subgraph "Subagent Lifecycle"
        Spawn[Spawn Subagent]
        Execute[Execute Task]
        Monitor[Monitor Progress]
        Collect[Collect Results]
    end

    subgraph "Task Management"
        TaskMgr[Tasks Manager]
        TaskList[Task List]
        Status[Task Status]
    end

    Handler --> TaskConfig
    Handler --> Executor

    Executor --> Plan
    Executor --> Explore
    Executor --> General

    Handler --> Spawn
    Spawn --> Execute
    Execute --> Monitor
    Monitor --> Collect

    Handler --> TaskMgr
    TaskMgr --> TaskList
    TaskMgr --> Status

    style Handler fill:#3498DB,color:#fff
    style Executor fill:#1ABC9C,color:#fff
```

### Prompt Management

```mermaid
graph LR
    subgraph "Prompt Manager"
        Manager[Prompt Manager]
        Builder[Prompt Builder]
    end

    subgraph "Prompt Sources"
        System[System Prompt]
        Recipe[Recipe Instructions]
        Extensions[Extension Instructions]
        Frontend[Frontend Instructions]
    end

    subgraph "Dynamic Components"
        Tools[Available Tools]
        Context[Session Context]
        Mode[Goose Mode]
        Hints[Goose Hints]
    end

    subgraph "Final Prompt"
        Combined[Combined Prompt]
    end

    Manager --> Builder

    Builder --> System
    Builder --> Recipe
    Builder --> Extensions
    Builder --> Frontend

    Builder --> Tools
    Builder --> Context
    Builder --> Mode
    Builder --> Hints

    System --> Combined
    Recipe --> Combined
    Extensions --> Combined
    Frontend --> Combined
    Tools --> Combined
    Context --> Combined
    Mode --> Combined
    Hints --> Combined

    style Manager fill:#8E44AD,color:#fff
    style Combined fill:#2ECC71,color:#fff
```

### Multi-Model (Lead-Worker) Pattern

```mermaid
sequenceDiagram
    participant User
    participant Agent
    participant Lead as Lead Model
    participant Worker as Worker Model

    User->>Agent: Complex Request

    Agent->>Agent: Check if Plan Mode

    alt Multi-Model Enabled
        Agent->>Worker: Generate Plan
        Worker-->>Agent: Task Plan
        Agent->>User: Show Plan
        User->>Agent: Approve Plan

        loop For Each Task
            Agent->>Lead: Execute Task
            Lead-->>Agent: Task Result
            Agent->>User: Update Progress
        end
    else Single Model
        Agent->>Lead: Direct Execution
        Lead-->>Agent: Result
    end

    Agent-->>User: Final Response
```

## Key Design Patterns

### 1. **Provider Abstraction**
- Trait-based abstraction for all LLM providers
- Unified streaming interface
- Format adapters for provider-specific protocols

### 2. **Extension via MCP**
- Protocol-based extension system
- Dynamic loading and hot-reload
- Isolated execution environments

### 3. **Event-Driven Architecture**
- Async/await throughout
- Stream-based communication
- Notification system for real-time updates

### 4. **Layered Security**
- Permission system for tool execution
- Security inspector for threat detection
- Malware scanning for extensions

### 5. **State Management**
- Session-based isolation
- Persistent conversation history
- Extension state management

### 6. **Modular Tool System**
- Dynamic tool registration
- Tool routing and indexing
- Built-in and extension tools

## Technology Stack

- **Language**: Rust
- **Async Runtime**: Tokio
- **Serialization**: serde, serde_json
- **HTTP Client**: reqwest
- **Streaming**: futures, async-stream
- **Protocol**: MCP (Model Context Protocol)
- **Error Handling**: anyhow, thiserror
- **Logging**: tracing, tracing-subscriber
- **Testing**: Standard Rust testing + scenario tests

## File Organization

```
crates/goose/src/
├── agents/              # Agent orchestration and tools
│   ├── agent.rs        # Main agent implementation
│   ├── extension_manager.rs
│   ├── mcp_client.rs
│   ├── prompt_manager.rs
│   ├── recipe_tools/   # Recipe-related tools
│   ├── subagent_execution_tool/
│   └── ...
├── providers/          # LLM provider integrations
│   ├── base.rs        # Provider trait
│   ├── factory.rs     # Provider factory
│   ├── anthropic.rs
│   ├── openai.rs
│   ├── formats/       # Format adapters
│   └── ...
├── conversation/       # Message and conversation management
│   ├── mod.rs
│   └── message.rs
├── session/           # Session management
│   ├── session_manager.rs
│   └── extension_data.rs
├── recipe/            # Recipe system
│   ├── mod.rs
│   └── build_recipe/
├── config/            # Configuration system
│   ├── base.rs
│   └── ...
├── context_mgmt/      # Context and token management
├── permission/        # Permission system
├── security/          # Security inspection
├── execution/         # Execution management
├── mcp_utils.rs      # MCP utilities
├── model.rs          # Model configuration
└── lib.rs            # Library entry point
```

## Related Documentation

- [CLAUDE.md](./CLAUDE.md) - Claude integration guide
- [CONTRIBUTING.md](./CONTRIBUTING.md) - Contributing guidelines
- [AGENTS.md](./AGENTS.md) - Agent concepts
- [Provider Documentation](https://block.github.io/goose/docs/getting-started/providers)
- [Recipe Documentation](https://block.github.io/goose/docs/guides/recipes/session-recipes)
- [MCP Documentation](https://modelcontextprotocol.io)

## Architecture Principles

1. **Modularity**: Clear separation of concerns with well-defined module boundaries
2. **Extensibility**: Plugin architecture via MCP for external integrations
3. **Security**: Defense in depth with multiple security layers
4. **Performance**: Async-first design with streaming support
5. **Reliability**: Robust error handling and retry mechanisms
6. **Observability**: Comprehensive tracing and diagnostics
7. **Flexibility**: Support for multiple providers, models, and configurations

## Future Considerations

- **Agent Coordination**: Enhanced multi-agent collaboration
- **Distributed Execution**: Cross-machine subagent execution
- **Advanced Planning**: Improved planning and reasoning capabilities
- **Memory System**: Long-term memory and knowledge management
- **Tool Marketplace**: Community tool sharing and discovery
- **Performance Optimization**: Caching, batching, and optimization strategies
