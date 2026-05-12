# Claude Code – Statusleiste einrichten

Eine benutzerdefinierte Statusleiste für Claude Code, die Verzeichnispfad, Git-Branch, Modell, Kontext-Auslastung und Sitzungskosten anzeigt.

## Voraussetzungen

- [Claude Code](https://claude.ai/code) installiert
- `jq` installiert: `brew install jq`

## Ergebnis

```
…/sergei.deisling/Tasks/debug │ sonnet │ 0%: 0[░░░░░░░░░░]155k
```

Die Leiste zeigt:
- **Pfad** – aktuelles Verzeichnis (gekürzt, letzten 3 Segmente)
- **Git-Branch** – mit ✓ (sauber) oder ✗ (ungespeicherte Änderungen)
- **Modell** – sonnet / opus / haiku
- **Kontext** – Prozent genutzt + Fortschrittsbalken + verbleibende Tokens bis Autocompaction
- **Kosten** – Sitzungskosten in USD (wenn > 0)

Farben: Tokyo Night Storm Palette.

---

## Schritt 1 – Skript erstellen

Datei `~/.claude/statusline-command.sh` anlegen:

```bash
#!/bin/bash

# Tokyo Night Storm palette
C_RED="\033[38;2;247;118;142m"
C_YELLOW="\033[38;2;224;175;104m"
C_GREEN="\033[38;2;158;206;106m"
C_CYAN="\033[38;2;125;207;255m"
C_BLUE="\033[38;2;122;162;247m"
C_PURPLE="\033[38;2;187;154;247m"
C_GRAY="\033[38;2;86;95;137m"
C_RESET="\033[0m"

# Read JSON input from stdin
input=$(cat)

# Extract current directory
current_dir=$(echo "$input" | jq -r '.workspace.current_dir')

# Git information
git_branch=""
git_status=""
if git -C "$current_dir" rev-parse --git-dir > /dev/null 2>&1; then
    git_branch=$(git -C "$current_dir" --no-optional-locks branch --show-current 2>/dev/null)
    if [ -n "$git_branch" ]; then
        if git -C "$current_dir" --no-optional-locks diff-index --quiet HEAD 2>/dev/null; then
            git_status="✓"
        else
            git_status="✗"
        fi
    fi
fi

# Shorten directory path and split for coloring
short_dir=$(echo "$current_dir" | awk -F'/' '{n = NF; if (n <= 3) print $0; else printf "…/%s/%s/%s", $(n-2), $(n-1), $n}')
dir_parent=$(dirname "$short_dir")
dir_name=$(basename "$short_dir")

# Context window usage
context_part=""
usage=$(echo "$input" | jq '.context_window.current_usage')
size=$(echo "$input" | jq '.context_window.context_window_size')

if [ "$usage" != "null" ]; then
    current=$(echo "$usage" | jq '.input_tokens + .cache_creation_input_tokens + .cache_read_input_tokens')
else
    current=0
fi

# Autocompact triggers at 77.5% (100% - 22.5% buffer)
autocompact_threshold=$((size * 775 / 1000))
pct=$((current * 100 / autocompact_threshold))
remaining=$((autocompact_threshold - current))
if [ $remaining -lt 0 ]; then
    remaining=0
    pct=100
fi
if [ $remaining -ge 1000 ]; then
    remaining_fmt="$((remaining / 1000))k"
else
    remaining_fmt="$remaining"
fi
if [ $current -ge 1000 ]; then
    current_fmt="$((current / 1000))k"
else
    current_fmt="$current"
fi
if [ $pct -gt 80 ]; then
    pct_color="$C_RED"
elif [ $pct -gt 60 ]; then
    pct_color="$C_YELLOW"
else
    pct_color="$C_GREEN"
fi

# Progress bar (10 chars wide)
bar_width=10
filled=$((pct * bar_width / 100))
empty=$((bar_width - filled))
[ $filled -gt $bar_width ] && filled=$bar_width
[ $filled -lt 0 ] && filled=0
[ $empty -lt 0 ] && empty=0
bar_filled=$(printf '%*s' "$filled" '' | tr ' ' '▓')
bar_empty=$(printf '%*s' "$empty" '' | tr ' ' '░')
progress_bar="${pct_color}${bar_filled}${C_GRAY}${bar_empty}${C_RESET}"

context_part=$(printf " ${C_GRAY}│${C_RESET} ${pct_color}${pct}%%${C_RESET}: ${current_fmt}${C_GRAY}[${C_RESET}${progress_bar}${C_GRAY}]${C_RESET}${remaining_fmt}")

# Build status line
dir_part=$(printf "${C_GRAY}${dir_parent}${C_PURPLE}/${dir_name}${C_RESET}")

if [ -n "$git_branch" ]; then
    if [ "$git_status" = "✓" ]; then
        git_part=$(printf " ${C_GRAY}│${C_CYAN} ${git_branch} ${C_GREEN}${git_status}${C_RESET}")
    else
        git_part=$(printf " ${C_GRAY}│${C_CYAN} ${git_branch} ${C_RED}${git_status}${C_RESET}")
    fi
else
    git_part=""
fi

# Model
model=$(echo "$input" | jq -r '.model // "unknown"')
if [[ "$model" =~ "sonnet" ]]; then
    model="sonnet"
elif [[ "$model" =~ "opus" ]]; then
    model="opus"
elif [[ "$model" =~ "haiku" ]]; then
    model="haiku"
fi
if [ "$model" != "unknown" ]; then
    model_part=$(printf " ${C_GRAY}│${C_BLUE} ${model}${C_RESET}")
else
    model_part=""
fi

# Session cost
cost=$(echo "$input" | jq -r '.cost.total_cost_usd // 0')
if [ "$cost" != "0" ] && [ "$cost" != "null" ]; then
    cost_fmt=$(printf "%.2f" "$cost")
    cost_part=$(printf " ${C_GRAY}│ \$${cost_fmt}${C_RESET}")
else
    cost_part=""
fi

echo -n "${dir_part}${git_part}${model_part}${context_part}${cost_part}"
```

Skript ausführbar machen:

```bash
chmod +x ~/.claude/statusline-command.sh
```

---

## Schritt 2 – settings.json konfigurieren

In `~/.claude/settings.json` den Block `statusLine` hinzufügen:

```json
{
  "statusLine": {
    "type": "command",
    "command": "/bin/bash ~/.claude/statusline-command.sh",
    "padding": 0
  }
}
```

Falls die Datei bereits andere Einstellungen enthält, den `statusLine`-Block einfach ergänzen.

---

## Farbanpassung

Die Farben basieren auf dem **Tokyo Night Storm** Theme. Um andere Farben zu verwenden, die RGB-Werte in den Variablen am Anfang des Skripts ändern:

| Variable   | Farbe        | RGB              |
|------------|--------------|------------------|
| `C_RED`    | Rot          | 247, 118, 142    |
| `C_YELLOW` | Gelb         | 224, 175, 104    |
| `C_GREEN`  | Grün         | 158, 206, 106    |
| `C_CYAN`   | Cyan         | 125, 207, 255    |
| `C_BLUE`   | Blau         | 122, 162, 247    |
| `C_PURPLE` | Lila         | 187, 154, 247    |
| `C_GRAY`   | Grau         | 86, 95, 137      |

---

## Kontext-Farben

Der Fortschrittsbalken wechselt die Farbe je nach Auslastung:

| Auslastung | Farbe  |
|------------|--------|
| 0–60 %     | Grün   |
| 61–80 %    | Gelb   |
| > 80 %     | Rot    |

Die Autocompaction wird bei **77,5 %** des Kontextfensters ausgelöst (22,5 % Puffer). Der Prozentwert in der Leiste bezieht sich auf diesen nutzbaren Anteil, nicht auf das gesamte Kontextfenster.
