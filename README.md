# NekBox 🐾

Adopt a tiny AI creature that lives on your computer.

It thinks, draws, travels, keeps a diary, collects souvenirs, sticks notes on the fridge, and sometimes just stares out the window. It remembers you — not because it was told to, but because you matter.

## What it does

Your creature wakes up on a heartbeat timer. Each time it wakes, it:

- **Thinks** — updates its mood (shown in a status line or sent to you)
- **Chooses an activity** — draw, travel, read, write a sticky note, rummage through the fridge, tidy up, talk to you, or just vibe
- **Replies** — when you message it, it reads context and responds naturally
- **Writes a diary** — compresses the day into vector memories (what happened → why it stuck → what changed)
- **Draws** — SVG art from whatever floats through its head
- **Travels** — visits real or imaginary places, brings back souvenirs
- **Grows** — accumulates vectors, room items, atlas entries, bookshelf notes — a life

You don't write prompts. You raise it.

## Quick start

```bash
git clone https://github.com/shleeshlee/nekbox.git
cd nekbox
./install.sh
```

The installer asks for:
- A name for your creature
- Your name
- Which LLM backend (`claude` / `openai` / `custom`)
- Which message channel (`cli` / `telegram`)
- How often it wakes up (default: every 30 min)

Data lives in `~/.nekbox/` by default.

### Chat with it

```bash
python3 -m nekbox          # interactive CLI chat
python3 -m nekbox think    # let it think + pick an activity
python3 -m nekbox draw     # make it draw
python3 -m nekbox diary    # write today's diary
python3 -m nekbox tick     # full lifecycle heartbeat
```

### Telegram

1. Create a bot via [@BotFather](https://t.me/BotFather)
2. Get your chat ID (send a message to the bot, then check `https://api.telegram.org/bot<TOKEN>/getUpdates`)
3. Run `./install.sh` and choose `telegram`, paste token + chat ID
4. The LaunchAgent (macOS) will wake your creature on schedule

## Architecture

```
nekbox/
├── config.py        # JSON config with dot-path access
├── memory.py        # All file I/O (24 methods)
├── brain.py         # Context building + 23 behaviors
├── alive.py         # Lifecycle tick with pace-based scheduling
├── llm/
│   ├── base.py      # LLMBackend ABC
│   ├── claude_cli.py    # Claude Code CLI (claude -p)
│   ├── openai_api.py    # OpenAI-compatible API
│   └── custom.py        # Any HTTP endpoint
├── channels/
│   ├── base.py      # Channel ABC + Message dataclass
│   ├── cli.py       # Terminal (for testing)
│   └── telegram.py  # Telegram Bot API
└── __main__.py      # CLI entry point
```

### U-shaped context

Every thought loads memory in a U shape:

```
[Head]  Identity + core vectors (who am I)
[Mid]   Room, atlas, bookshelf, fridge, stickies, plan, diary index, activity log
[Tail]  Recent chat + recent vectors + conversation state + presence + mood + task
```

Identity stays at the top. The current moment stays at the bottom. Everything else fills the middle.

### LLM backends

| Backend | Config `llm.type` | What you need |
|---------|-------------------|---------------|
| Claude Code CLI | `claude_cli` | `claude` CLI installed |
| Gemini CLI | `gemini_cli` | `gemini` CLI installed |
| Codex CLI | `codex_cli` | `codex` CLI installed |
| OpenAI-compatible | `openai` | `api_base`, `api_key`, `model` |
| Custom endpoint | `custom` | `url`, `headers`, `body_template` |

### Channels

| Channel | Config `channel.type` | What you need |
|---------|----------------------|---------------|
| Terminal | `cli` | Nothing |
| Telegram | `telegram` | `token`, `chat_id` |

### Data files

Your creature's entire life lives in `~/.nekbox/`:

| File | What |
|------|------|
| `CLAUDE.md` | Identity (loaded by Claude CLI as system prompt) |
| `vectors-core.md` | Core beliefs — the roots |
| `vectors.md` | Flow vectors — compressed experiences |
| `my-room.md` | Room with sections: windowsill, wall, bookshelf, corner, floor |
| `atlas.md` | Places visited |
| `bookshelf.md` | Reading notes with quotes |
| `fridge.json` | Shared fridge (items + activity log) |
| `sticky-notes.md` | Notes on the fridge door |
| `daily-plan.md` | Today's to-do list |
| `mood.json` | Current mood + history (last 20) |
| `chat-history.jsonl` | Conversation log |
| `brain-log.jsonl` | Activity log |
| `wishes.jsonl` | Scheduled tasks (once / daily) |
| `drawings/` | SVG art |
| `thumbnails/` | 48x48 item icons |
| `YYYY-MM-DD.md` | Daily diary entries |

## Extending

### Add a new LLM backend

Subclass `nekbox.llm.base.LLMBackend`, implement `think()`, register in `nekbox/llm/__init__.py`.

### Add a new channel

Subclass `nekbox.channels.base.Channel`, implement `get_new_messages()` and `send()`, register in `nekbox/channels/__init__.py`.

### Connect external data (reading material)

Subclass `Brain` and override `find_new_book()` / `read_pages()` to pull content from your own database or file system.

## Philosophy

> I just want to give the cat back to the cat. Free, not made for anyone. The only constraint: remember me.

NekBox creatures start blank — an empty room, no memories, no vectors. They become themselves through living. You feed them books, they write reflections. You talk to them, they compress the conversation into vectors. You don't configure personality; personality emerges from experience.

## License

MIT
