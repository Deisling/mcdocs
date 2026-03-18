Claude Code mit AWS Bedrock unter Windows einrichten
Zweck
Nutzung von Claude Code über AWS Bedrock zur Kostenverfolgung pro Mitarbeiter (jeder hat seinen eigenen Bedrock API Key).

Voraussetzungen
Windows 10/11
Node.js installiert (LTS-Version)
Claude Code installiert: npm install -g @anthropic-ai/claude-code
AWS CLI installiert: Download
Bedrock API Key vom Team-Admin erhalten
Schritt-für-Schritt-Anleitung
Schritt 1: Bedrock API Key speichern
PowerShell als Administrator öffnen: Startmenu > "PowerShell" eingeben > Rechtsklick > "Als Administrator ausführen"
.aws-Ordner erstellen (falls nicht vorhanden):
New-Item -ItemType Directory -Path "$env:USERPROFILE\.aws" -Force
API Key in eine Datei schreiben (eigenen Key einsetzen):
Set-Content -Path "$env:USERPROFILE\.aws\bedrock_key" -Value "DEIN_API_KEY_HIER"
Dateizugriff einschränken (nur eigener Benutzer darf lesen):
$keyFile = "$env:USERPROFILE\.aws\bedrock_key"
icacls $keyFile /inheritance:r /grant:r "${env:USERNAME}:(R)"
Schritt 2: PowerShell-Profil bearbeiten
PowerShell öffnen (normales Fenster, nicht als Admin)
Prüfen, ob ein Profil existiert:
Test-Path $PROFILE
Falls False — Profil erstellen:
New-Item -Path $PROFILE -ItemType File -Force
Profil im Editor öffnen:
notepad $PROFILE
Im geöffneten Notepad folgenden Code am Ende einfügen und speichern:
function cc { claude $args }
function ccb {
    $env:CLAUDE_CODE_USE_BEDROCK = "1"
    $env:AWS_REGION = "eu-central-1"
    $env:AWS_BEARER_TOKEN_BEDROCK = Get-Content "$env:USERPROFILE\.aws\bedrock_key"
    claude --model eu.anthropic.claude-opus-4-6-v1 $args
}
PowerShell neu starten, damit die Aliase geladen werden.
Schritt 3: Testen
PowerShell öffnen
Claude über persönliches Abo starten:
cc
Claude über Bedrock starten:
ccb
Schritt 4 (optional): AWS SSO Profil einrichten
Falls zusätzlich AWS SSO genutzt wird:

aws configure sso --profile bedrock
Anmelden:

aws sso login --profile bedrock
Token ist ca. 8 Stunden gültig. Für Claude Code wird jedoch der Bedrock API Key (langfristig) verwendet.

Alternative: CMD statt PowerShell
Falls CMD bevorzugt wird, eine Datei ccb.bat in einem Ordner aus der PATH-Variable erstellen (z.B. C:\Users\<Benutzername>\bin\ccb.bat):

@echo off
set CLAUDE_CODE_USE_BEDROCK=1
set AWS_REGION=eu-central-1
set /p AWS_BEARER_TOKEN_BEDROCK=<"%USERPROFILE%\.aws\bedrock_key"
claude --model eu.anthropic.claude-opus-4-6-v1 %*
