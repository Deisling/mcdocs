---
date: 2026-03-18
tags: [claude, aws, bedrock, setup, windows]
---

# Claude Code mit AWS Bedrock unter Windows einrichten

## Zweck

Nutzung von Claude Code über AWS Bedrock zur Kostenverfolgung pro Mitarbeiter (jeder hat seinen eigenen Bedrock API Key).

## Voraussetzungen

- Windows 10/11
- [Node.js](https://nodejs.org/) installiert (LTS-Version)
- Claude Code installiert: `npm install -g @anthropic-ai/claude-code`
- AWS CLI installiert: [Download](https://aws.amazon.com/cli/)
- Bedrock API Key vom Team-Admin erhalten

## Schritt-für-Schritt-Anleitung

### Schritt 1: Bedrock API Key speichern

1. **PowerShell als Administrator öffnen**: Startmenu > "PowerShell" eingeben > Rechtsklick > "Als Administrator ausführen"
2. `.aws`-Ordner erstellen (falls nicht vorhanden):
   ```powershell
   New-Item -ItemType Directory -Path "$env:USERPROFILE\.aws" -Force
   ```
3. API Key in eine Datei schreiben (eigenen Key einsetzen):
   ```powershell
   Set-Content -Path "$env:USERPROFILE\.aws\bedrock_key" -Value "DEIN_API_KEY_HIER"
   ```
4. Dateizugriff einschränken (nur eigener Benutzer darf lesen):
   ```powershell
   $keyFile = "$env:USERPROFILE\.aws\bedrock_key"
   icacls $keyFile /inheritance:r /grant:r "${env:USERNAME}:(R)"
   ```

### Schritt 2: PowerShell-Profil bearbeiten

1. **PowerShell öffnen** (normales Fenster, nicht als Admin)
2. Prüfen, ob ein Profil existiert:
   ```powershell
   Test-Path $PROFILE
   ```
3. Falls `False` — Profil erstellen:
   ```powershell
   New-Item -Path $PROFILE -ItemType File -Force
   ```
4. Profil im Editor öffnen:
   ```powershell
   notepad $PROFILE
   ```
5. **Im geöffneten Notepad** folgenden Code am Ende einfügen und speichern:
   ```powershell
   function cc { claude $args }
   function ccb {
       $env:CLAUDE_CODE_USE_BEDROCK = "1"
       $env:AWS_REGION = "eu-central-1"
       $env:AWS_BEARER_TOKEN_BEDROCK = Get-Content "$env:USERPROFILE\.aws\bedrock_key"
       claude --model eu.anthropic.claude-opus-4-6-v1 $args
   }
   ```
6. **PowerShell neu starten**, damit die Aliase geladen werden.

### Schritt 3: Testen

1. PowerShell öffnen
2. Claude über persönliches Abo starten:
   ```powershell
   cc
   ```
3. Claude über Bedrock starten:
   ```powershell
   ccb
   ```

### Schritt 4 (optional): AWS SSO Profil einrichten

Falls zusätzlich AWS SSO genutzt wird:
```powershell
aws configure sso --profile bedrock
```

Anmelden:
```powershell
aws sso login --profile bedrock
```

Token ist ca. 8 Stunden gültig. Für Claude Code wird jedoch der Bedrock API Key (langfristig) verwendet.

### Alternative: CMD statt PowerShell

Falls CMD bevorzugt wird, eine Datei `ccb.bat` in einem Ordner aus der PATH-Variable erstellen (z.B. `C:\Users\<Benutzername>\bin\ccb.bat`):
```bat
@echo off
set CLAUDE_CODE_USE_BEDROCK=1
set AWS_REGION=eu-central-1
set /p AWS_BEARER_TOKEN_BEDROCK=<"%USERPROFILE%\.aws\bedrock_key"
claude --model eu.anthropic.claude-opus-4-6-v1 %*
```

## Model ID in Bedrock

| Modell | Model ID |
|--------|----------|
| Opus 4.6 | `anthropic.claude-opus-4-6-v1` |
| Sonnet 4.6 | `anthropic.claude-sonnet-4-6` |

Für Cross-Region-Inference wird ein Regionspräfix hinzugefügt: `eu.anthropic.claude-opus-4-6-v1`
