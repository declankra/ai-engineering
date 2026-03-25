# AI Engineering

Encoding hard-won learnings on building AI products in an AI-native way.

These are opinionated, interactive tools — skills for AI coding agents — that help you build AI products that actually work. Not demos. Not prototypes. Products that ship.

---

## Skills

- **[`/autoresearch-setup`](./autoresearch/)** — Interactive guide that determines whether your AI product needs a self-improving optimization loop, and if it does, walks you through setting one up. Identifies your riskiest AI surface, verifies you have real data, helps you define the right metric, builds your program file, and scaffolds the full loop infrastructure. Opinionated — it will tell you no if you're not ready.

---

## What is this?

AI products fail in a specific way: the demo works, the last 10% doesn't, and you can't tell why. The gap is almost never the model's capability — it's the scaffolding around it.

These skills encode a methodology for closing that gap:

1. **Pick the smallest slice** that delivers a core value prop and tests one core risk
2. **Get verifiable data** — if the AI's outputs can't be checked against reviewed truth, nothing downstream works
3. **Build a benchmark** that exercises live product behavior, not mocks
4. **Run a scientific loop** — hypothesis, bounded change, measure, keep or discard
5. **Bake in your domain knowledge** — the model is capable, but it needs your judgment on what "correct" means

The skills automate the parts that can be automated and make sure the parts that need your judgment get it.

## Install

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

This started from applying [Karpathy's autoresearch loop](https://github.com/karpathy/autoresearch) to real product development — specifically, optimizing vendor quote extraction accuracy from 76% to 97% in a construction procurement tool. The method works, but the setup is where most people get stuck. These skills help with the setup.

Read the full writeup: [Building a Self-improving Agent Loop for AI Products](https://declankramper.com/writes/building-a-self-improving-agent-loop-for-ai-quote-extraction)

## Contributing

If you try this and learn something, open an issue. The skills get better when they encode more real-world failure modes.

## License

MIT
