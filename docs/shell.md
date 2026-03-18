---
date: 2026-03-16
tags: [claude, aws, bedrock, setup, macos]
---

# Claude Code mit AWS Bedrock einrichten (macOS)

## Zweck

Nutzung von Claude Code über AWS Bedrock zur Kostenverfolgung pro Mitarbeiter (jeder hat seinen eigenen Bedrock API Key).

## Was konfiguriert wurde

### 1. Aliase in `~/.zshrc`

```bash
alias cc="claude"     # über persönliches Abonnement
alias ccb='CLAUDE_CODE_USE_BEDROCK=1 AWS_REGION=eu-central-1 AWS_BEARER_TOKEN_BEDROCK=$(cat ~/.aws/bedrock_key) claude --model eu.anthropic.claude-opus-4-6-v1'
```

- `cc` — Start über persönliches Abonnement (Max/Pro)
- `ccb` — Start über das Unternehmens-AWS-Bedrock

### 2. Bedrock API Key

Gespeichert in `~/.aws/bedrock_key` (chmod 600). Jeder Mitarbeiter verwendet seinen eigenen Schlüssel zur Kostenverfolgung.

### 3. AWS SSO Profil (optional)

Zusätzlich wurde das Profil `bedrock` über `aws configure sso` eingerichtet. Anmeldung:

```bash
aws sso login --profile bedrock
```

Token ist ca. 8 Stunden gültig. Für Claude Code wird jedoch der Bedrock API Key (langfristig) verwendet.

### 4. Model ID in Bedrock

| Modell | Model ID |
|--------|----------|
| Opus 4.6 | `anthropic.claude-opus-4-6-v1` |
| Sonnet 4.6 | `anthropic.claude-sonnet-4-6` |

Für Cross-Region-Inference wird ein Regionspräfix hinzugefügt: `eu.anthropic.claude-opus-4-6-v1`

## Statusline

Das Skript `~/.claude/statusline-command.sh` funktioniert auch mit Bedrock — es zeigt Modell, Kontext und Sitzungskosten an.
