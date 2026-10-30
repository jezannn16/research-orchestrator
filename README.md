![Research Orchestrator cover](assets/cover.png)

# Research Orchestrator

[English](README.md) · [Русский](README.ru.md) · [Español](README.es.md)

**Stop collecting tabs. Start connecting evidence.**

A reusable agent skill that takes a question from a focused lookup to a cited investigation across the web, GitHub, documents, academic literature and connected internal sources.

## What it does

Quick fact checks; deep web comparisons; GitHub code, issues and releases; and research that combines multiple source types.

## How it works

Decomposes complex questions into distinct angles, delegates where supported, then combines findings with citations, source conflicts and gaps.

## Requirements

Research tools and, for delegated modes, an agent runtime that supports subagents. Connected sources are used only when available.

## Install

```bash
mkdir -p ~/.agents/skills
git clone https://github.com/jezannn16/research-orchestrator.git ~/.agents/skills/research-orchestrator
```

## Try it

> Compare the leading open-source options for [problem], cite the evidence, and identify unresolved gaps.

See [SKILL.md](SKILL.md) and [references/](references/) for the complete workflow.
