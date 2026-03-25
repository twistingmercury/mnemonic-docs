# Authoring Patterns

This guide teaches you how to write patterns that work well with semantic search, follow Mnemonic conventions, and deliver value to your team.

## What is a Pattern?

A pattern is a Markdown document with YAML frontmatter that captures a piece of team knowledge: a coding convention, architectural decision, best practice, or technique. Patterns are loaded into Mnemonic, semantically enriched (embedded), and served to AI coding assistants via MCP search.

When Claude Code or another AI assistant searches Mnemonic with a question, it finds patterns by semantic similarity — understanding the concepts you're asking about, not just keyword matching. Well-written patterns are discoverable, precise, and actionable.

## Pattern File Structure

Every pattern file has three parts:

1. **YAML Frontmatter** (metadata)
2. **H1 Title** (the pattern name as a heading)
3. **Body** (content sections, some decorated for indexing)

Here's the overall shape:

```markdown
---
[frontmatter fields here]
---

# [Pattern Title]

## Overview

[Overview section - never indexed]

[//]: pattern
## [Section 1]

[Section 1 content - indexed]

[//]: pattern
## [Section 2]

[Section 2 content - indexed]
```

## Frontmatter Schema

### Required Fields

| Field | Type | Format | Constraints |
| --- | --- | --- | --- |
| `name` | string | kebab-case | Machine identifier. Regex: `^[a-z][a-z0-9-]*$`. Max 128 characters. Must be unique. |
| `entity_type` | string | kebab-case | Category. Examples: `best-practice`, `cli-pattern`, `api-specification`, `architectural-decision`, `technique`. |
| `language` | string | enum | One of the supported languages (see below). |
| `domain` | string | enum | One of the supported domains (see below). |
| `description` | string | free text | Non-empty. Used for search and UI display. Max 500 characters. Should be a complete sentence or short paragraph. |

### Supported Language Values

`agnostic`, `bash`, `c`, `cpp`, `csharp`, `cql`, `cypher`, `dart`, `delphi`, `docker`, `elixir`, `erlang`, `go`, `json`, `java`, `javascript`, `kotlin`, `lua`, `markdown`, `matlab`, `mql`, `objective-c`, `perl`, `php`, `plsql`, `powershell`, `python`, `r`, `react`, `ruby`, `rust`, `scala`, `shell`, `sql`, `swift`, `terraform`, `toml`, `tsql`, `typescript`, `visual-basic`, `yaml`, `zig`.

Use `agnostic` for patterns that apply across languages (e.g., code review practices, documentation standards).

### Supported Domain Values

`api-design`, `backend`, `frontend`, `testing`, `devops`, `cli`, `data-design`, `documentation`, `data-access`, `security`, `shell-scripting`, `configuration`, `observability`, `source-management`.

Choose the primary domain where the pattern applies most directly. If your pattern spans multiple domains, pick the most specific one.

### Optional Fields

| Field | Type | Notes |
| --- | --- | --- |
| `agents` | array of string | Names of agents this pattern is relevant to (e.g., `go-software-engineer`, `bats-test-engineer`). Enrichment may discover and add more. When loaded via the API, these are mapped to `agent_associations` with a default relevance of `0.8`. See [Understanding the Agents Field](#understanding-the-agents-field) below. |
| `tags` | array of string | Search keywords for filtering (e.g., `error-handling`, `cobra-cli`, `integration-testing`). Keep these specific and actionable. |
| `version` | string | Language, framework, or spec version. Examples: `Go 1.21+`, `React 18+`, `Postgres 14+`. |
| `related_patterns` | array of string | Names of related patterns (using the `name` field of those patterns). Enrichment may discover and add more. |

### Understanding the Agents Field

In AI coding assistants like Claude Code, **agents** are specialist subagents that handle specific types of work. A team typically defines a set of agents tailored to their stack and workflows — for example, a `go-software-engineer` agent that writes Go code, a `bats-test-engineer` that creates shell script tests, or a `devops-engineer` that manages Docker and CI/CD pipelines. Each agent has its own tools, permissions, and domain expertise.

The `agents` field in a pattern's frontmatter declares which of these specialists the pattern is most relevant to. When an agent searches Mnemonic, patterns tagged with that agent's name are weighted higher in results, making it more likely the agent finds the guidance it needs.

For example, a pattern about Go error handling might list:

```yaml
agents:
  - go-software-engineer
  - go-e2e-test-engineer
```

This tells Mnemonic that both the Go engineer and the Go test engineer should see this pattern when searching for error-handling guidance.

You don't need to list every possible agent — enrichment can discover additional associations automatically. Focus on the agents that would benefit most from the pattern.

For a real-world example of how teams define agents and skills, see the [mnemonic-agents-skills](https://github.com/twistingmercury/mnemonic-agents-skills) repository.

### Unknown Fields Are Allowed

You can add custom metadata fields (e.g., `author`, `created`, `status`). Unknown fields are ignored by validation and by Mnemonic's API, but they're preserved for your own tooling and documentation.

## The `[//]: pattern` Decorator

The pattern decorator is the key mechanism that controls what gets indexed for semantic search.

### How It Works

Place `[//]: pattern` on its own line immediately before a heading (`##`, `###`, or deeper):

```markdown
[//]: pattern
## How We Handle Errors

Everything from this heading until the next decorator or end of file
is indexed as a searchable chunk.
```

### What Gets Indexed

| Content | Indexed? | Why |
| --- | --- | --- |
| Decorated section (heading + body) | Yes | Marked explicitly for search |
| `## Overview` section | No | Intentionally excluded; provides context, not search result |
| Intro prose before first decorator | No | Supports reading but not searching |
| Undecorated section | No | Not marked for indexing |
| Empty decorated body | No | Silently dropped |

### Why Decorators Matter

Decorators let you control search relevance. Content that's important for humans reading the file (introductions, changelogs, disclaimers) can be stored without polluting search rankings. This keeps search results focused on actionable guidance.

### Special Case: Overview Section

The `## Overview` section should never be decorated with `[//]: pattern`. By convention, Overview provides context for human readers — what the pattern is, when to use it, and why it matters — without competing in search rankings. If you omit the decorator (as you should), the Overview is not indexed.

### Multiple Decorated Sections

If your pattern covers multiple independent techniques, use multiple decorated sections:

```markdown
[//]: pattern
## Setup and Initialization

Content about setup...

[//]: pattern
## Error Handling

Content about error handling...

[//]: pattern
## Performance Tuning

Content about performance...
```

Each decorated section becomes an independently searchable chunk. Someone searching "Go performance tuning" will find the third section directly, without noise from the setup section.

## Required Sections

Every pattern must have:

1. **`## Overview`** — What the pattern is, when to use it, and why. Keep this concise (a few sentences to a short paragraph). It's your elevator pitch.

2. **At least one `[//]: pattern` decorated section** — The actual knowledge. Without decorated sections, there's nothing to search.

## Writing Patterns for Semantic Search

Semantic search works by understanding concepts, not matching keywords. Here are practices that make patterns discoverable and useful.

### 1. Make Each Decorated Section Independently Discoverable

Someone searching "how to wrap errors in Go" should find your error-handling section without needing surrounding context. Each section should make sense on its own.

Bad:

```markdown
[//]: pattern
## Implementation Details

All error handling logic uses the fmt.Errorf wrapper.
```

Good:

```markdown
[//]: pattern
## Wrapping Errors with fmt.Errorf

When propagating errors up the call stack, wrap them with fmt.Errorf and %w
to preserve the error chain. This allows callers to use errors.Is and errors.As
for inspection.

Example:
```

The good version includes what the pattern is, why you'd use it, and an example. A search for "error wrapping" will find it.

### 2. Use Descriptive Headings

The section heading becomes part of the searchable content. Make it descriptive, not generic.

Bad: `## Implementation`, `## Example`, `## Details`

Good: `## Wrapping Errors`, `## Mocking Database Calls in Unit Tests`, `## Performance Tuning for High-Throughput Topics`

### 3. Include Code Examples

Embeddings capture the semantics of code. A pattern with concrete examples is more discoverable than one with only prose. Show, don't just tell.

````markdown
[//]: pattern
## Error Wrapping with fmt.Errorf

When propagating errors up the call stack, use fmt.Errorf with %w:

```go
if err != nil {
    return fmt.Errorf("failed to load config: %w", err)
}
```

This preserves the error chain. Callers can inspect the underlying error:

```go
err := doWork()
if errors.Is(err, os.ErrNotExist) {
    // handle missing file
}
```
````

### 4. Write for AI Context

The pattern content is what Claude Code reads. Make it actionable and explicit.

- "Do this, not that" with examples
- Explain the trade-offs
- Include error cases or exceptions
- Provide concrete syntax and patterns

Avoid vague guidance ("be careful," "consider," "might want to"). AI assistants work better with specificity.

### 5. Keep Chunks Focused

A decorated section covering one technique is better than one covering five. Focused sections rank better in semantic search and are easier for an AI to apply.

Bad:

```markdown
[//]: pattern
## Configuration Management

Our CLI uses Viper for configuration loading, with defaults from config.yaml,
overrides from environment variables, and CLI flags taking precedence. We also
use Cobra for command structure and custom middleware for validation...
```

Good:

```markdown
[//]: pattern
## Configuration Loading with Viper

Define a Config struct and load it with Viper. Use explicit function parameters
instead of package-level globals.

[code example]

[//]: pattern
## CLI Flag Binding

Use Cobra's BindFlag to connect CLI flags to Viper keys.

[code example]
```

### 6. Use Tags for Precision

Tags enable conjunctive (AND) filtering in search. Choose tags that are specific and actionable.

Good tags: `error-handling`, `cobra-cli`, `integration-test`, `postgres-migration`, `jwt-auth`

Avoid vague tags: `code`, `pattern`, `tech`, `important`

## Complete Example

Here's a full, realistic pattern:

`````markdown
---
name: cobra-configuration-pattern
entity_type: cli-pattern
language: go
domain: cli
description: Configuration management with explicit config injection, environment variable overrides, and clear precedence rules for Cobra CLIs
agents:
  - go-software-engineer
tags:
  - cobra
  - cli
  - configuration
  - viper
  - environment-variables
version: Go 1.21+
related_patterns:
  - cobra-root-command-pattern
  - error-handling-in-go
---

# Cobra Configuration Pattern

## Overview

This pattern demonstrates configuration management for Cobra CLIs with explicit
config passing, environment variable overrides, and clear precedence rules.

The pattern separates configuration concerns: defaults come from code, files
provide overrides, environment variables can be used in deployment scenarios,
and CLI flags take final precedence.

[//]: pattern
## Define Configuration Structure

Create a Config struct to hold all configuration values. Keep the struct minimal
and focused — only include settings your CLI actually needs.

```go
type Config struct {
    Host       string
    Port       int
    LogLevel   string
    Timeout    time.Duration
    Workers    int
}
```

Load configuration from multiple sources using Viper, with explicit precedence:
file → environment variables → CLI flags.

```go
func LoadConfig(cfgFile string) (*Config, error) {
    v := viper.New()

    // 1. Set defaults
    v.SetDefault("host", "localhost")
    v.SetDefault("port", 8080)
    v.SetDefault("log_level", "info")
    v.SetDefault("timeout", 30*time.Second)
    v.SetDefault("workers", 4)

    // 2. Load from file if provided
    if cfgFile != "" {
        v.SetConfigFile(cfgFile)
        if err := v.ReadInConfig(); err != nil {
            return nil, fmt.Errorf("failed to load config file: %w", err)
        }
    }

    // 3. Override with environment variables
    v.SetEnvPrefix("MYAPP")
    v.AutomaticEnv()

    // 4. Unmarshal into struct
    var cfg Config
    if err := v.Unmarshal(&cfg); err != nil {
        return nil, fmt.Errorf("failed to parse config: %w", err)
    }

    return &cfg, nil
}
```

[//]: pattern
## Bind CLI Flags

Use Cobra's built-in flag binding to allow CLI flags to override configuration.
Bind flags after loading the config file but before unmarshaling.

```go
func setupFlags(cmd *cobra.Command, cfg *Config) {
    cmd.Flags().String("host", cfg.Host, "Server hostname")
    cmd.Flags().Int("port", cfg.Port, "Server port")
    cmd.Flags().String("log-level", cfg.LogLevel, "Log level")

    // Bind flags to Viper keys
    viper.BindPFlag("host", cmd.Flags().Lookup("host"))
    viper.BindPFlag("port", cmd.Flags().Lookup("port"))
    viper.BindPFlag("log_level", cmd.Flags().Lookup("log-level"))
}
```

Bind flags in your command's PreRunE or PersistentPreRunE to ensure they're
applied before command execution.

[//]: pattern
## Passing Config Explicitly

Never use package-level globals for configuration. Pass config as a method
receiver or function parameter:

```go
type Server struct {
    cfg *Config
}

func NewServer(cfg *Config) *Server {
    return &Server{cfg: cfg}
}

func (s *Server) Run(ctx context.Context) error {
    // Use s.cfg.Host, s.cfg.Port, etc.
}
```

This makes testing easier and config flow explicit. A reader of your code can
immediately see where config comes from.

[//]: pattern
## Example: Complete CLI with Configuration

Here's a minimal but complete example:

```go
package main

import (
    "context"
    "fmt"
    "github.com/spf13/cobra"
    "github.com/spf13/viper"
)

type Config struct {
    Host    string
    Port    int
    Workers int
}

func loadConfig(cfgFile string) (*Config, error) {
    v := viper.New()
    v.SetDefault("host", "localhost")
    v.SetDefault("port", 8080)
    v.SetDefault("workers", 4)

    if cfgFile != "" {
        v.SetConfigFile(cfgFile)
        if err := v.ReadInConfig(); err != nil {
            return nil, err
        }
    }

    v.SetEnvPrefix("MYAPP")
    v.AutomaticEnv()

    var cfg Config
    if err := v.Unmarshal(&cfg); err != nil {
        return nil, err
    }
    return &cfg, nil
}

type App struct {
    cfg *Config
}

func (a *App) Run(ctx context.Context) error {
    fmt.Printf("Starting server on %s:%d with %d workers\n",
        a.cfg.Host, a.cfg.Port, a.cfg.Workers)
    return nil
}

func main() {
    var cfgFile string

    rootCmd := &cobra.Command{
        Use:   "myapp",
        Short: "My application",
        RunE: func(cmd *cobra.Command, args []string) error {
            cfg, err := loadConfig(cfgFile)
            if err != nil {
                return err
            }

            app := &App{cfg: cfg}
            return app.Run(cmd.Context())
        },
    }

    rootCmd.Flags().StringVar(&cfgFile, "config", "", "Config file path")
    rootCmd.Flags().String("host", "localhost", "Server host")
    rootCmd.Flags().Int("port", 8080, "Server port")

    viper.BindPFlag("host", rootCmd.Flags().Lookup("host"))
    viper.BindPFlag("port", rootCmd.Flags().Lookup("port"))

    rootCmd.Execute()
}
```

Usage:

```bash
# Use defaults
./myapp

# Override via environment variable
MYAPP_HOST=0.0.0.0 ./myapp

# Override via CLI flag
./myapp --host 0.0.0.0 --port 9000

# Load from config file
./myapp --config /etc/myapp/config.yaml
```
`````

## Validation

The mnemonic-patterns repository includes a validation script that checks all patterns for correctness:

```bash
./install/validate.sh --dir ./patterns
```

### What Validation Checks

1. YAML frontmatter is present and parseable
2. All required fields are present and non-empty
3. `name` matches the regex `^[a-z][a-z0-9-]*$`
4. `language` is one of the supported enum values
5. `domain` is one of the supported enum values
6. `entity_type` uses kebab-case
7. `description` is non-empty
8. At least one `[//]: pattern` decorator is present
9. `## Overview` section is present

### Dependencies

The validation script requires:

- `yq` (mikefarah/yq) — YAML parsing
- `jq` — JSON parsing

Install these in your environment before running validation.

### Validation Failures

If validation fails, the script outputs clear error messages with line numbers and field names. Common issues:

- Missing required fields: Add them to frontmatter
- Invalid enum values: Check your language or domain against the supported values list
- Missing Overview: Add a `## Overview` section
- No decorated sections: Add at least one `[//]: pattern` section
- Invalid name format: Use kebab-case only (no underscores, spaces, or caps)

## Best Practices

### Do

- Write patterns about specific, reusable techniques
- Include concrete code examples
- Keep sections focused (one technique per section)
- Use the Overview to set context
- Tag patterns to enable precise filtering
- Reference related patterns in frontmatter
- Write for both humans and AI assistants

### Don't

- Create patterns for one-off solutions or project-specific code
- Write vague guidance ("be careful," "as needed")
- Mix multiple unrelated techniques in one section
- Put important guidance outside decorated sections
- Use generic or vague tags
- Index the Overview section
- Leave patterns without examples

## What's Next

**[Managing Patterns](06-managing-patterns.md)** — Learn how to load patterns into Mnemonic, bulk-load from files, keep your knowledge base organized, and handle versioning and deprecation.

---

Copyright (c) 2025-2026 Jeremy K. Johnson. All rights reserved.
