---
title: 10 macOS Terminal Tricks You Never Knew Existed
description: Discover 10 powerful macOS terminal tricks that will make you a more
  efficient developer, including commands to open files in Finder, use Spotlight,
  keep your Mac awake, and more. This article is a follow-up to the author's previous
  post on shell tricks and shows you how to unlock the full potential of your Mac's
  terminal.
categories:
- programming
tags:
- MacOS
- Terminal
- Shell
- Spotlight
- Power Management
date: 2026-10-24
draft: false
showToc: true
---

# I Ignored These Mac Terminal Commands for Years <br> What most Mac developers never learn

Original appeared in [Medium](https://medium.com/@TusharKanjariya/i-ignored-these-mac-terminal-commands-for-years-6b18665d468d?sk=15fde4b79c810a798d0d23324ce1f40b)

[Tushar Kanjariya](https://medium.com/@TusharKanjariya?source=post_page---byline--6b18665d468d-----------------------------------------)

Jun 29, 2026


I once watched a developer kick off a 20-minute build, lock his screen, and walk away to make coffee.

When the build finished, his Mac *spoke* Out loud. “Build complete.”

He didn’t have some paid notification app. He didn’t have a custom setup that took a weekend to configure.

He had appended four words to his build command using a tool that has shipped with every Mac for over twenty years.

That was the day I realized something embarrassing: I had been using my Mac’s terminal as if it were a Linux box.

## The Problem Most Mac Developers Have

In my last post, [I Used the Terminal Wrong for Years](https://medium.com/@TusharKanjariya/i-used-the-terminal-wrong-for-years-5557b10c0b85?sk=e0d80926e44933dc968c582220e11e73), I shared the shell tricks I use every day like !!, Ctrl+R, aliases, fzf, tmux.

And here’s the thing about all of those: they’re *shell* tricks. They work the same on Ubuntu, on a remote server, on your Mac.

That’s their strength. It’s also why most Mac developers never go further.

Meanwhile, macOS ships with a layer of commands that hook directly into the operating system like Spotlight, Finder, the display server, the speech engine, the image pipeline and most developers never touch them.

Your shell is portable.

Your Mac is not.

There are commands that only make sense here, and they’re the ones nobody teaches you.

These are the ones I actually use. Not 50 commands.

Just the ones that stick.

## Trick #1: `open` — Stop Alt-Tabbing to Finder

The single most-used macOS command in my day, and most developers don’t know it exists.

    open .

That opens the current folder in Finder.

You’re in some deeply nested project directory in your terminal, and you need to drag a file somewhere open . and Finder appears exactly where you are.

But it goes much further:

```bash
open index.html              # open in default browser
open -a Safari index.html    # open in a specific app
open -a "Visual Studio Code" .
open https://github.com      # URLs work too
open -R config.json          # reveal the file in Finder (highlighted)
```

The `-R` flag is the underrated one. It doesn’t open the file but it opens Finder with that file selected. Perfect for “where on earth did that build output go.”

I used to keep a Finder window open and manually navigate it to match my terminal. For years. open . made me feel a little stupid and a lot faster.

## Trick #2: `mdfind` — Spotlight Lives in Your Terminal

You know how find works: it crawls the filesystem, directory by directory, and on a big drive it takes forever.

Your Mac already has an index of every file on the disk. It’s called Spotlight, and you’ve been using it with Cmd+Space.

What nobody tells you: it has a command-line interface.

    mdfind "docker-compose.yml"

Results are instant. Not fast but instant. It’s a database lookup, not a filesystem crawl.

Scope it to the current folder:

    mdfind -onlyin . "TODO"

And because Spotlight indexes content, not just filenames, that command finds files containing the word TODO even including inside PDFs and documents.

Find by name only:

    mdfind -name ".zshrc"

The first time you run mdfind after years of waiting on find /, it feels like a cheat code.

## Trick #3: `caffeinate` — Your Mac Stays Awake Exactly As Long As It Needs To

You’re running a long deploy over SSH. You walk away. Your Mac goes to sleep. The connection drops. The deploy dies.

The usual fix is changing your energy settings and then forgetting to change them back, and your laptop cooks itself in your bag.

The right fix:

    caffeinate ./deploy.sh

Your Mac stays awake while the command runs and the moment it finishes, normal sleep behavior returns. No settings to remember. No cleanup.

Need it open-ended? `caffeinate -t 3600` keeps the Mac awake for an hour. Plain caffeinate keeps it awake until you Ctrl+C.

This is the kind of tool that’s invisible until the day it saves a 40-minute deploy, and after that you never run a long process without it.

## Trick #4: `say` — Let Your Mac Tell You When It’s Done

This is the trick from the opening, and it’s two characters longer than you’d expect:

    npm run build && say "build done" || say "build failed"

Start the build. Go make coffee. Your Mac announces the result to the room.

It sounds like a toy. It is not a toy. The real problem it solves: every developer alt-tabs to check on a long-running process like tests, builds, downloads every five, ten, fifteen times.
Each check is a context switch. `say` deletes all of them.

You can even pick the voice:

    say -v "Daniel" "deployment finished"

People on calls with me have heard my Mac announce test results mid-conversation. Every single one of them asked how.

## Trick #5: `screencapture` — Screenshots Without Touching the Mouse

You know Cmd+Shift+4. Here’s its scriptable older sibling:

```bash
screencapture -c            # full screen → straight to clipboard
screencapture -i shot.png   # interactive selection → file
screencapture -T 5 shot.png # 5-second timer (open that dropdown menu first)
screencapture -w window.png # click a window, capture just it
```

The `-c` flag is the daily driver where full screenshot directly to the clipboard, no file cluttering your desktop, ready to paste into Slack.

And the `-T` timer flag solves the classic impossible screenshot: menus and dropdowns that close the moment you press a keyboard shortcut. Run it with a delay, open the menu, wait.

## Trick #6: `sips` — Delete ImageMagick, It Was Built In the Whole Time

This one hurt when I learned it.

I have installed ImageMagick on every Mac I’ve ever owned. Every one. To resize the occasional image, convert a PNG to JPEG, nothing fancy.

macOS ships with sips (Scriptable Image Processing System)

```sh
sips -Z 800 photo.png                      # resize to max 800px (keeps ratio)
sips -s format jpeg photo.png --out photo.jpg
sips -r 90 photo.png                       # rotate 90 degrees
sips -Z 200 *.png                          # batch resize an entire folder
```

That last one is the killer. Forty screenshots that need to become thumbnails? One line, in-place, done before ImageMagick would have finished installing.

Warning from experience: sips modifies files in place unless you use
--out. Copy first, resize second.

## Trick #7: `textutil` — The Document Converter Nobody Knows Exists

Someone sends you a .docx. You need its text in your terminal, or in Markdown, or as HTML for a quick page.
The usual path involves opening Word or uploading the file to some sketchy “free converter” website.
The built-in path:

```sh
textutil -convert txt report.docx     # docx → plain text
textutil -convert html report.docx    # docx → html
textutil -convert docx notes.html     # and back the other way
textutil -cat txt *.docx -output all.txt   # merge many docs into one text file
```

It reads and writes txt, html, rtf, doc, docx, and more. Offline. Instant. Shipped with your Mac since before some of your dependencies existed.
The `-cat` variant is sneaky useful: feed it a folder of meeting-notes documents and get one searchable text file out.

## Trick #8: `defaults write` — The Settings Apple Didn’t Put in System Settings

macOS has hundreds of settings with no UI. They’re all reachable from the terminal.
The two I set on every new Mac within the first hour:

```sh
# Screenshots: stop dumping them on the Desktop
mkdir -p ~/Screenshots
defaults write com.apple.screencapture location ~/Screenshots

# Screenshots as JPEG instead of huge PNGs
defaults write com.apple.screencapture type jpg
```

And the famous ones:

```sh
# Show hidden files in Finder
defaults write com.apple.finder AppleShowAllFiles -bool true
killall Finder

# Speed up dock animation
defaults write com.apple.dock autohide-time-modifier -float 0.3
killall Dock
```

A new Mac setup that used to take me an afternoon of clicking through settings is now a script I run once.

That’s the real unlock here defaults makes your machine configuration reproducible, like infrastructure.

## Trick #9: `networkQuality` — A Speed Test Without the Ad-Covered Website

“Is it my WiFi or is the server slow?” the eternal debugging question.
You don’t need a browser tab for this anymore:

    networkQuality

Your Mac runs a full speed test from the terminal like download, upload, latency, and a responsiveness score (that last one measures how your connection behaves under load, which is what actually matters on video calls).

Add `-v` for detailed output. It’s been built in since macOS Monterey, and I’d estimate roughly nobody knows about it.

## Trick #10: `pmset` — Ask Your Mac Why It Woke Up at 3 AM

`pmset` controls and inspects power management. Two uses earn it a place on this list.
The quick one is battery status in scripts and tmux status bars:

    pmset -g batt

The detective one:

    pmset -g log | grep -i wake

That shows you why your Mac has been waking up and every wake event and what caused it.
Mysterious battery drain overnight? The answer is in that log.
Mine turned out to be a Bluetooth device politely waking the machine every 40 minutes, all night, every night.

## Bonus: `qlmanage -p` — Quick Look From the Terminal

In Finder, you press Space on a file and get an instant preview. In the terminal:

    qlmanage -p diagram.png

Same Quick Look preview, no Finder needed. Press Space or Esc to close it and you’re back at your prompt. Perfect for “is this the right image?” without opening Preview and then closing Preview.

It’s technically a Quick Look debugging tool, which is exactly why no tutorial ever mentions it.

## The Real Shift

Here’s what changed for me: I stopped thinking of macOS as “Linux with a nicer screen.”

The shell tricks from my last post are the portable layer they follow you to every server you’ll ever SSH into.

But your Mac has a second layer underneath: Spotlight’s index, the speech engine, the image pipeline, the power log. All of it scriptable. All of it sitting there since the day you unboxed the machine.

None of these take setup. There’s nothing to install that’s the entire point. The tools were already there. The only thing missing was knowing their names.

Start with one. `open .` if you’ve been alt-tabbing to Finder. `say` if you babysit long builds. `mdfind` if you’ve ever waited on find.

The terminal is a tool. But on a Mac, it’s a bigger tool than anyone told you.

Thanks for reading 🙏


🤖 created with help of ai

