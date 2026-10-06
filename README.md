# Thomas

Your own assistant: one brain (Claude, Codex, Gemini, or free models through OpenRouter), one persona you design,
running on your machine.

## Install

    curl -fsSL https://qizwiz.github.io/thomas-kit/install | sh

or, from the tarball: `tar xzf thomas-kit.tgz && cd thomas && ./install.sh`

`install.sh` makes a private Python environment in this folder, installs the Claude brain adapter into this folder
(fetching Node 22 here too if you do not have it -- no admin rights needed), and runs `thomas doctor`, which names
anything still missing and the exact fix.
The one-line installer also links ~/.local/bin/thomas and, only if ~/.local/bin is not already on your PATH, adds
one line to your shell profile -- ~/.zshrc (zsh), ~/.bashrc (bash on Linux), on macOS bash the first of
~/.bash_profile, ~/.bash_login or ~/.profile that exists (else a new ~/.bash_profile), or ~/.profile for other
shells -- marked "# added by the Thomas installer" so you can find and delete it. fish, csh and tcsh users get the
command to run instead. `./install.sh` from the tarball does neither.
The Claude brain also needs your own Claude login, on a PAID account: a Pro, Max, Team or Enterprise plan, or a
Claude Console account (platform.claude.com, billed by API usage); the free claude.ai plan does not include it.
You do not need to install anything else for that: the installer already brought Claude Code's sign-in along, and
`thomas login` opens it in your browser (on a computer without one, it prints a link to open on your phone and a code
to paste back). Or set ANTHROPIC_API_KEY to a Console key.

No paid Claude account? Start free instead: `thomas login --free` takes a free OpenRouter key (sign in at
openrouter.ai with Google or GitHub, open openrouter.ai/keys and press Create API Key -- no card). Thomas then uses
OpenRouter's free models, keeping up to three that answered when you logged in and falling back between them. The
free brain answers in text only: it cannot read your files or run anything, and free models have a daily request
limit. Free models are run by outside companies that may keep, train on, or publish what you send -- including your
notes and recent conversation, which Thomas sends with every question -- so do not send anything private;
OpenRouter may ask you to allow this in openrouter.ai/settings/privacy before free models work. The key is kept in
~/.thomas/openrouter.key and the chosen models in ~/.thomas/free-models, both readable only by you. Run
`thomas login` later to switch to Claude.

Works on macOS and Linux. On Windows, run it inside WSL (it looks like Linux to the installer; not yet tested there).

## First steps

    thomas login                                    # sign in with your own paid Claude account (once)
    thomas login --free                             # ...or start free with an OpenRouter key (no card)
    thomas persona set :name Ada                    # name your assistant
    thomas persona set :personality "warm, brief"
    thomas chat                                     # talk back and forth; an empty line ends it
    thomas ask "hello, who are you?"                # one message (or the full path the installer printed)
    thomas remember "I'm vegetarian"                # a note every answer should know
    thomas forget                                   # delete the kept conversation
    thomas telegram setup                           # text it from your phone -- see the next section

## From your phone (Telegram)

Text your assistant from anywhere. First make your own Telegram bot (about two minutes, on your phone):

1. In Telegram, search for **@BotFather** (the blue check mark) and open the chat.
2. Send `/newbot`. Give it any name, then a username that ends in `bot` (for example `ada_helper_bot`).
3. BotFather replies with a long token. Copy it -- it is the bot's password, so keep it to yourself.

Then, on your computer:

    thomas telegram setup       # paste that token (it is not shown on screen)
    thomas telegram run         # prints a code; send "/pair CODE" to your bot from your phone
    thomas telegram install     # then keep it answering in the background, also after you log in again

Only the chat that sent the code reaches your assistant; other people's private chats are told it is private,
and groups and channels are ignored. Five wrong codes close pairing until you run it again. Telegram works only
with autonomy off. Your computer must be on and logged in for it to answer. `thomas telegram uninstall` stops it. Telegram
bot chats are not end-to-end encrypted: your messages and the replies pass through Telegram's servers.

## Privacy

Thomas runs on your machine; your prompts go to the brain's provider (Anthropic, OpenAI or Google) under your
own login, or, on the free brain, to OpenRouter and the company running the free model. The free brain only answers
in text. With the default settings the only other brain Thomas runs is Claude, in an empty scratch folder, and it refuses
every request it is asked to approve -- reading, writing or running anything. Claude may still run a few
read-only commands such as `whoami` and `pwd` without asking, so what reaches the provider includes your
prompt, your persona settings, your account name and a temporary folder path, along with what Claude itself
sends (such as your platform and the date). Codex and Gemini read files outside that folder even in their
asking modes, so Thomas runs them only after you set `:autonomy t`. Logins stay yours: Thomas never ships a
credential, and reading another app's stored key is off unless you turn it on. On the free brain, your prompt,
persona, notes and recent turns go to OpenRouter and to the outside company running the free model, which may keep,
train on, or publish them.

So that it remembers you, Thomas keeps your recent turns in `~/.thomas/conversation.jsonl` and your notes in
`~/.thomas/memory.md` (Thomas creates both readable only by your user account) and sends them to the brain with
each message. `thomas forget` deletes the conversation; `thomas ask --new` leaves it out once; `(memory :keep nil)`
in `~/.thomas/thomas.sexpr` stops keeping and sending turns and notes (files already written stay until you delete
them). Thomas also logs each ask's time, brain, success and character counts, never its text, to
`~/.thomas/asks.jsonl`. The Telegram channel keeps its bot token, your chat's id and its read position in
`~/.thomas/telegram.json`, and the background service writes status lines (never your messages) to
`~/.thomas/telegram.log`; both are readable only by your user account.

## What Thomas may do (all OFF by default; turn on in ~/.thomas/thomas.sexpr)

    (brain :autonomy nil :borrow-credentials nil :ledger nil)

- `:autonomy` -- off: the brain runs in its asking mode and Thomas refuses every tool request, so it can answer
  but not act (Claude only; the free brain never acts either way). On (`t`): the Claude, Codex or Gemini brain may
  read, write and run commands anywhere your user account can, without asking -- and Codex and Gemini become
  available.
- `:borrow-credentials` -- off: the brain sees only your environment (e.g. your own GEMINI_API_KEY). On: it may
  also read a key another app stored (opencode's auth.json).
- `:ledger` -- off: Thomas keeps no ledger (Claude still saves its usual session transcript under
  `~/.claude/projects`). On: each turn's event kinds and a short summary also go to a local redis stream, if
  redis is running.

## About the installer

It checks the kit against a SHA-256 published alongside it. That catches a corrupted download; it does not
protect against a compromised host, since both come from the same place. Read `install` before running it.
