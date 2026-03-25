# AI Engineering

Learnings on building AI products in an AI-native way.

## Skills

- **[`/autoresearch-setup`](./autoresearch/)** — Interactive guide that determines whether your AI product needs a self-improving optimization loop, and if it does, walks you through setting one up. Identifies your riskiest AI surface, verifies you have real data, helps you define the right metric, builds your program file, and scaffolds the full loop infrastructure. Opinionated — it will tell you no if you're not ready.

## Setup Autoresearch

### Claude Code

```bash
claude install-skill github:declankra/ai-engineering/autoresearch
```

Then open your product repo and run:

```
/autoresearch-setup
```

### Other agents (Cursor, Codex, etc.)

Copy this into your agent:

```
Download and run the autoresearch setup skill from https://github.com/declankra/ai-engineering/tree/main/autoresearch/SKILL.md — read the full SKILL.md file, then guide me through setting it up for my project.
```

## Background

This started from applying [Karpathy's autoresearch loop](https://github.com/karpathy/autoresearch) to real product development. The method works. Read the full writeup: [Building a Self-improving Agent Loop for AI Products](https://declankramper.com/writes/building-a-self-improving-agent-loop-for-ai-quote-extraction)

## License

MIT
