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

```bash
claude plugin add devmonks-diagrams@devmonks/ai-toolkit
```

## Usage

Invoke directly:

```
/d2 system architecture for a chat app with WebSocket, Redis pub/sub, and PostgreSQL
```

Or describe what you need and Claude will invoke the skill automatically:

> "Draw an ERD for users, posts, and comments"
> "Diagram the authentication flow"
> "Create a sequence diagram for the checkout process"

## Supported diagram types

- System architecture
- ERD (Entity-Relationship)
- Flowchart
- Sequence diagram
- Class diagram
- Grid layout

## How it works

1. Writes a `.d2` source file based on your description
2. Validates all icon URLs via curl
3. Compiles to SVG using the D2 CLI
4. Reports the output path

## License

MIT
