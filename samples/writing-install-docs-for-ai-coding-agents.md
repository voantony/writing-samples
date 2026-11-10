# Writing install docs for AI coding agents

An install guide for coding agents has to say two things a README usually doesn't: which steps the agent may run, and which ones it must hand back to a human. Put the never-do rules first, label every human step with a check the agent can run afterwards, and test the real behaviour rather than a status flag.

More people now install developer tools by pasting a repo link into a coding agent and saying "set this up." That works well for a pure command-line tool. It goes badly when part of the install can only be done by the human, which on macOS is common. When I shipped kindle-mtp-mount, a small tool that shows a USB Kindle as a drive in Finder, I wrote an AGENTS.md alongside the README. This is what I learned writing it.

## Why can't the agent just follow the README?

kindle-mtp-mount uses macFUSE's FSKit backend and libmtp. To get it working, someone has to:

1. Install Homebrew and libmtp.
2. Install macFUSE, which asks for a password and an approval prompt.
3. Turn on macFUSE's file system extension in System Settings.
4. Quit Calibre, because only one program can hold the Kindle at a time.
5. Build, test and install the tool.

| Step | Who does it | How the agent knows it's done |
| --- | --- | --- |
| 1. Homebrew and libmtp | Agent (asks the human for Homebrew if missing) | `command -v brew`; `install.sh` checks libmtp |
| 2. Install macFUSE | Human | `/Library/Filesystems/macfuse.fs` exists |
| 3. Enable the file system extension | Human | The read-only mount succeeds |
| 4. Quit Calibre | Human | The mount no longer reports the Kindle as busy |
| 5. Build, test, install | Agent | `install.sh` finishes its dry check |

An agent can do 1 and 5. Steps 2–4 belong to the person at the keyboard. A capable agent left to itself will try to "help" with them anyway, by poking at security settings, suggesting Recovery Mode changes or resetting the USB connection when the device looks busy. Each of those is somewhere between annoying and harmful.

## Why put the rules before the steps?

The file opens with rules, not steps. They're short and absolute:

- Never change System Settings, privacy or security settings, and never install system or kernel extensions.
- Never tell the human to lower security in Recovery Mode. It isn't needed with FSKit.
- Don't use sudo.
- Don't write to the Kindle except inside a scratch folder you create, and remove it afterwards.
- Don't reset or "fix" the USB connection. If the Kindle is busy, stop and tell the human.
- Keep the device serial masked when you report logs.

Putting them first means the agent reads them before it starts planning. Each one closes off a specific mistake I could picture an agent making.

## How should the doc mark human steps?

In the step list, every step the person has to do is labelled "(human step)" and says what to ask for and when to wait. Step 2 says: if `/Library/Filesystems/macfuse.fs` doesn't exist, ask the human to install macFUSE 5.x, "Don't install macFUSE for them." The agent gets a file path to check, so it isn't guessing whether the step is done.

## How does the agent know a step worked?

The trickiest case was the System Settings switch. There's a command-line tool, `pluginkit`, that looks like it should report whether the extension is on. In practice it may not show it as enabled even when it works. So the doc tells the agent not to rely on it: "The mount in step 6 is the real test." Step 6 mounts read-only and lists the folders, and if the mount fails with "File system extension not enabled," the agent knows exactly which step to go back to.

That pattern held up throughout. A test that exercises the real thing beats a status check that's only usually right.

## What happens when the agent retries?

Agents retry. `install.sh` is written to be safe to run any number of times. It checks for each requirement, installs libmtp if needed, builds, runs the unit tests against a fake in-memory device, installs to `~/.local/bin` and does a dry check. It never uses sudo and never installs macFUSE. When something is missing it stops with one sentence saying what the human needs to do, which the agent can pass along word for word.

## How can an agent use a GUI app?

The menu-bar app registers a URL scheme, so scripts and agents can run `open mountkindle://mount`, `open mountkindle://unmount` and `open mountkindle://quit`. It costs very little to add, and it means an agent can use the app without trying to click on menus.

## Takeaways

- Put the never-do rules at the top.
- Label human steps and give a file or command the agent can check afterwards.
- Pick verification steps that test the real behaviour.
- Make the installer idempotent and its errors one-line instructions.
- Give the app a scriptable entry point.
