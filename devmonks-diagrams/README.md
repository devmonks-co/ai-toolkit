# devmonks-diagrams

Generate diagrams from natural language using [D2](https://d2lang.com).

## Prerequisites

Install the D2 CLI:

```bash
# macOS
brew install d2

# Linux / other
curl -fsSL https://d2lang.com/install.sh | sh -
```

## Install

From your terminal:

```bash
claude plugin install devmonks-diagrams@ai-toolkit
```

Or from inside a Claude Code session:

```
/plugin install devmonks-diagrams@ai-toolkit
```

## Usage

| Command | Purpose | Arguments |
|---|---|---|
| `/d2` | Generate a D2 diagram from a natural language description | description of the diagram |

Or describe what you need and Claude will invoke the skill automatically:

> "Draw an ERD for users, posts, and comments"
> "Diagram the authentication flow"
> "Create a sequence diagram for the checkout process"

### Supported diagram types

- System architecture
- ERD (Entity-Relationship)
- Flowchart
- Sequence diagram
- Class diagram
- Grid layout

### How it works

1. Writes a `.d2` source file based on your description
2. Validates all icon URLs via curl
3. Compiles to SVG using the D2 CLI
4. Reports the output path

## License

MIT
