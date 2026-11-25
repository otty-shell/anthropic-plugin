# otty-plugin-anthropic

The official Anthropic plugin of OTTY. OTTY has no code for any agent by
name: this plugin teaches it Claude Code. Its agent part `claude-code`
names Claude Code's default config folder (`~/.claude`, or the one
`CLAUDE_CONFIG_DIR` names; the user adds other folders, such as
`~/.claude-work`, as profiles), tells whether each profile is logged in,
lists its sessions and reads the tokens of every answer with its session
and project. A profile whose `settings.json` sends Claude Code to another
server, such as DeepSeek's, has its tokens counted for that server's
host. The plugin never sees a key or a token: OTTY hands it handles in
their place.

Its provider part `anthropic` finds the Anthropic accounts itself, in the
Claude Code profile folders OTTY gives it, and reads only there while
OTTY asks. A Claude login is a subscription, known by its account and
named by its email (and its organization, unless it is the personal one
Claude makes for every account), with its plan; it stays the same account
when Claude Code renews its tokens, and one account logged in to two
profiles is one account found in two places. A login to an Anthropic
Console organization that pays per token is that organization. An
organization's name keeps no parentheses, since OTTY writes the name in
parentheses after "Anthropic": "Example Org (Claude API Platform)" is
named "Example Org, Claude API Platform". A Console
key and an Anthropic key of `settings.json` are keys, named by the end of
their fingerprint; an admin key is an organization. A key that
`settings.json` sends to another server, such as DeepSeek's, is no
Anthropic account.

Anthropic tells nothing of what a key spent, so a key or a Console
organization gets "Your spend this month" at API prices: the answers of
this calendar month in the Claude Code sessions of the folders it pays
for now, priced at the price OTTY gives during `numbers`, kind by kind.
It counts only this computer's Claude Code sessions, never OTTY's own
login folder, and leaves out answers sent to another server. A bill read
with an admin key comes first.

It reads the numbers of an account any way it can, only in the folders
the account was found in: the limits of a Claude login (and its extra
usage) from Anthropic's usage request with the login's token, and the
cost report of an organization with its own admin key or the one the
user gives for it, both from `api.anthropic.com`. When the folder holds
no token, as when Claude Code keeps it elsewhere, or the server gives no
answer, it reads the limits Claude Code itself shows: it starts `claude`
pointed at that folder, unable to run anything, in a terminal nobody
sees, types `/usage` and reads the panel from the screen. It answers
Claude Code's question whether to trust a folder only for OTTY's own
probe folder, and afterwards deletes the session files that run left in
`projects/<probe folder>/` of that folder; its manifest allows it to
delete nothing else (`write` of its `logins` entry, inside the folders
of the call only). OTTY runs this at most once in five minutes per
account. With no way that works it reads nothing, which is no failure.
It renews no token itself: Claude Code renews it when it runs. OTTY's
own login (Add account → "Log in with Claude Code") is read first, and
when its token is past its time or Anthropic refused it, the `/usage`
probe runs there, so Claude Code keeps that login fresh; when even that
fails, the account needs "Log in again".

What only the user can do, the parts ask for themselves, in OTTY's
language, which they follow when the user picks another: `claude` in a
profile nobody is logged in with, and in the folder of a login that
expired with no way to renew it or that Anthropic refused (Claude Code
logs in or renews the login), and a field for the admin key
(`sk-ant-admin…`) when the bill of an organization needs one or Anthropic
refused the one given.

The native installer of Claude Code puts the program at
`~/.local/bin/claude`, a folder the login shell may not have on its
`PATH`: the manifest lists it in `program_places`, so OTTY finds the
program there and starts it by that path, also for `/usage`. When
Claude Code is nowhere, the manifest's `install` gives its native
installer, `curl -fsSL https://claude.ai/install.sh | bash`, which OTTY
runs in a terminal tab for **Install Claude Code**, and its setup page,
`https://code.claude.com/docs/en/setup`.

## Build

The plugin takes `otty-plugin-sdk` from the `agents` branch of the OTTY
repository (`Cargo.toml`). The first build pins the SDK in `Cargo.lock`
to the newest commit of that branch; commit the lock, and run `cargo
update -p otty-plugin-sdk` to take a newer SDK. Its tests run natively
over the made-up files of `fixtures/`, which stand for the home folder:

```sh
rustup target add wasm32-wasip2
cargo test
cargo build --release --target wasm32-wasip2
```

The component is `target/wasm32-wasip2/release/otty_plugin_anthropic.wasm`.

To build against an OTTY checkout instead, such as one next to this
folder, point cargo at its SDK for one command, without changing
`Cargo.toml`:

```sh
cargo --config 'patch."https://github.com/otty-shell/otty".otty-plugin-sdk.path="../otty/crates/plugins/sdk"' test
```

## Install

In OTTY, open Plugins mode, press **Add plugin…**, paste
`https://github.com/otty-shell/anthropic-plugin`, then **Next** and
**Install**. OTTY installs the newest release it can run and offers the
later ones as updates.

To try a build that is not released, put the component as
`anthropic.wasm` in a folder with `otty-plugin.toml` and the icons
(`claude-code.svg` and `anthropic.svg`), and install that folder with
**Add plugin from folder…**. After a change, copy the new build there
and press **Reload** on the plugin's row.

## Release

Set the same `version` in `otty-plugin.toml` and `Cargo.toml`, commit,
and push the tag `v<version>`:

```sh
git tag v0.2.18
git push origin v0.2.18
```

The workflow `.github/workflows/release.yml` checks that the tag equals
`version` of `otty-plugin.toml`, runs the tests, builds the plugin for
`wasm32-wasip2` and attaches to the GitHub release of the tag the files
OTTY installs: `otty-plugin.toml`, `anthropic.wasm` and the icons. OTTY
refuses a release whose manifest names another version than its tag.
