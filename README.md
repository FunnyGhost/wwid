# 🔎 wwid — *What Was I Doing*

> find & resume Claude Code chats from anywhere

`wwid` indexes every [Claude Code](https://claude.com/claude-code) conversation
on your machine and lets you fuzzy-search them by name, PR, project, or first
prompt — then re-opens the one you pick with `claude --resume`, in its original
folder.

It is **read-only** by default — it only reads Claude's transcript files under
`~/.claude/projects/` — with one opt-in exception, `wwid --rename`, which names
un-named chats (and backs them up first). See *Renaming* below.

Looks like this 👇:
<img width="1823" height="610" alt="CleanShot 2026-06-02 at 16 41 12" src="https://github.com/user-attachments/assets/905f02be-54bb-4894-acfd-d31b43ebc03b" />
<img width="1825" height="540" alt="CleanShot 2026-06-02 at 16 40 55" src="https://github.com/user-attachments/assets/48b8195a-7931-454d-8a7b-0d878bd9392e" />

## 📦 Install

```bash
brew install FunnyGhost/tap/wwid
```

That single command also installs the `fzf` and Python dependencies. For the
latest commit on `main` instead of the last release:

```bash
brew install --HEAD FunnyGhost/tap/wwid
```

### Requirements

| Tool          | Why                                | Provided by                       |
|---------------|------------------------------------|-----------------------------------|
| `python3`     | runs the script (stdlib only)      | the formula (`python@3.13`)       |
| `fzf`         | the interactive picker             | the formula                       |
| Claude Code   | resume target + `--rename` titles  | `brew install --cask claude-code` |

`fzf` and Python are pulled in automatically. Claude Code is the one thing you
need yourself — `wwid` tells you if it's missing.

## 🧭 Usage

```bash
wwid                 # fuzzy picker over all chats; Enter resumes the one you pick
wwid <query>         # same picker, pre-filtered by <query>
wwid pr <num|url>    # find chat(s) linked to a PR; resume if there's exactly one
wwid list            # plain text table (good for scripts / grep)
wwid --rename        # name every chat you never named, from its content
wwid -h | --help     # usage
```

### In the picker

- **Type** to fuzzy-filter across every visible column **and** the (hidden)
  full first prompt.
- **Enter** resumes the highlighted chat — `wwid` `cd`s into the chat's
  original folder, then hands off to `claude --resume <id>`.
- **Esc** cancels without doing anything.

### Columns

| Column         | Meaning                                                            |
|----------------|-------------------------------------------------------------------|
| `NAME`         | your `/rename` name → else the auto AI title → else 1st prompt     |
| `AGE`          | time since the chat was last active                                |
| `CTX`          | approx % of the context window used at the last turn               |
| `MSGS`         | total user + assistant messages                                   |
| `PR`           | pull request(s) linked to the chat (auto-linked on `gh pr create`)|
| `PROJECT`      | the folder the chat ran in                                        |
| `FIRST PROMPT` | the first meaningful thing you typed                              |

### Renaming un-named chats (`wwid --rename`)

`wwid` is read-only **except** for this one command. It gives a short
(4–5 word) content-based name to every chat you never named yourself:

- **Skips** any chat that already has a `/rename` name, and any chat that's
  currently running.
- Generates each title with a cheap, side-effect-free `claude -p` call
  (Haiku, no saved session, no MCP, no tools).
- Writes the name by appending a `custom-title` record — exactly what
  `/rename` does — so it sticks and shows in Claude's `/resume` picker too.
- **Backs up** every file it touches to `~/.claude/.wwid-rename-backup-<ts>/`
  before writing, and asks for confirmation first.

```bash
wwid --rename --dry-run   # preview the new names, write nothing
wwid --rename             # do it (prompts before writing)
wwid --rename --yes       # do it without the confirmation prompt
```

To undo: restore the matching files from the backup folder (the title is the
last line of each touched transcript).

## 🛠️ How it works

Claude Code stores each conversation as a `.jsonl` file (one JSON record per
line) under `~/.claude/projects/<encoded-folder>/<session-id>.jsonl`. `wwid`
reads a few record types from each file:

| Record `type`  | Used for                          |
|----------------|-----------------------------------|
| `custom-title` | your manually set name            |
| `ai-title`     | the auto-generated name           |
| `pr-link`      | linked PR number / URL / repo     |
| `user`         | the first real prompt + cwd       |
| `assistant`    | message count + token usage (CTX) |

A full scan of all transcripts takes a fraction of a second, so `wwid` reads
them live on every run — no cache or index file to maintain.

The script itself (`bin/wwid`) is a single, heavily commented Python file with
zero third-party dependencies — open it to read how each piece works.

## 🧑‍💻 Development

```bash
git clone https://github.com/FunnyGhost/wwid.git
./wwid/bin/wwid --help          # run it straight from the checkout
brew install --HEAD FunnyGhost/tap/wwid   # or install your working main
```

The formula lives in [FunnyGhost/homebrew-tap](https://github.com/FunnyGhost/homebrew-tap).
To cut a release: tag it, `gh release create vX.Y.Z`, then bump `url` + `sha256`
in the tap's `Formula/wwid.rb`.

## 📝 License

MIT — see [LICENSE](LICENSE). © Catalin Ciubotaru.
