# Thomas

Your own assistant: one brain (Claude, Codex or Gemini), one persona you design, running on your machine.

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
The Claude brain also needs your own Claude login, on a PAID account: Claude Code needs a Pro, Max, Team or
Enterprise plan, or a Claude Console account (platform.claude.com, billed by API usage); the free claude.ai plan
does not include it.
Install Claude Code (https://claude.com/claude-code) and run `claude auth login`, or set ANTHROPIC_API_KEY.

Works on macOS and Linux. On Windows, run it inside WSL (it looks like Linux to the installer; not yet tested there).

## First steps

    thomas persona set :name Ada                    # name your assistant
    thomas persona set :personality "warm, brief"
    claude auth login                               # your own login (needs Claude Code) -- Thomas never ships one
    thomas chat                                     # talk back and forth; an empty line ends it
    thomas ask "hello, who are you?"                # one message (or the full path the installer printed)
    thomas remember "I'm vegetarian"                # a note every answer should know
    thomas forget                                   # delete the kept conversation

## Privacy

Thomas runs on your machine; your prompts go to the brain's provider (Anthropic, OpenAI or Google) under your
own login. With the default settings Thomas runs only the Claude brain, in an empty scratch folder, and refuses
every request it is asked to approve -- reading, writing or running anything. Claude may still run a few
read-only commands such as `whoami` and `pwd` without asking, so what reaches the provider includes your
prompt, your persona settings, your account name and a temporary folder path, along with what Claude itself
sends (such as your platform and the date). Codex and Gemini read files outside that folder even in their
asking modes, so Thomas runs them only after you set `:autonomy t`. Logins stay yours: Thomas never ships a
credential, and reading another app's stored key is off unless you turn it on.

So that it remembers you, Thomas keeps your recent turns in `~/.thomas/conversation.jsonl` and your notes in
`~/.thomas/memory.md` (Thomas creates both readable only by your user account) and sends them to the brain with
each message. `thomas forget` deletes the conversation; `thomas ask --new` leaves it out once; `(memory :keep nil)`
in `~/.thomas/thomas.sexpr` stops keeping and sending turns and notes (files already written stay until you delete
them). Thomas also logs each ask's time, brain, success and character counts, never its text, to
`~/.thomas/asks.jsonl`.

## What Thomas may do (all OFF by default; turn on in ~/.thomas/thomas.sexpr)

    (brain :autonomy nil :borrow-credentials nil :ledger nil)

- `:autonomy` -- off: the brain runs in its asking mode and Thomas refuses every tool request, so it can answer
  but not act (Claude only). On (`t`): the brain may read, write and run commands anywhere your user account
  can, without asking -- and Codex and Gemini become available.
- `:borrow-credentials` -- off: the brain sees only your environment (e.g. your own GEMINI_API_KEY). On: it may
  also read a key another app stored (opencode's auth.json).
- `:ledger` -- off: Thomas keeps no ledger (Claude still saves its usual session transcript under
  `~/.claude/projects`). On: each turn's event kinds and a short summary also go to a local redis stream, if
  redis is running.

## About the installer

It checks the kit against a SHA-256 published alongside it. That catches a corrupted download; it does not
protect against a compromised host, since both come from the same place. Read `install` before running it.
