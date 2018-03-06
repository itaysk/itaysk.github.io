---
title: Making my terminal look like VSCode
date: 2018-03-06 22:15
categories: [Technical-Howto]
tags: [tmux, vscode, iterm2]
---

Not sure how I went down this rabbit hole, but I ended up creating a VSCode inspired theme for iTerm2 and tmux (VSCode default dark theme).

## TL;DR

Source here:  
[https://gist.github.com/itaysk/6957eab43957c83f56554ffdcced4100](https://gist.github.com/itaysk/6957eab43957c83f56554ffdcced4100)

Result looks like this:  
![result](/images/2018-03-06-making-my-terminal-look-like-vscode_1.png)



## Finding VSCode's colors

Using the [Theme Color Reference](https://code.visualstudio.com/docs/getstarted/theme-color-reference), I found the colors in the source at:

- [theme.ts](https://github.com/Microsoft/vscode/blob/master/src/vs/workbench/common/theme.ts)
- [dark_defaults.json](https://github.com/Microsoft/vscode/blob/master/extensions/theme-defaults/themes/dark_defaults.json)

Some interesting pointers at:

- editor.background
- editor.foreground
- statusBar.background
- statusBar.foreground
- statusBarItem.activeBackground
- editorGroup.border

Note that for the `statusBarItem.activeBackground` style, there's a transparancy factor. Since tmux doesn't support colors with alpha channel, I found the matching opaque color hex for that shade.

There's also the ANSI basic colors palette which is redefined at:

- [terminalColorRegistry.ts](https://github.com/Microsoft/vscode/blob/92edecc7acd2694f4bc6fa68f08d971fc4721487/src/vs/workbench/parts/terminal/electron-browser/terminalColorRegistry.ts)

## Tmux

tmux has some nice configuration options around it's look and feel. Just search for `colour` in the man page and you'll find the following settings:

- window-active
- window
- pane-border
- pane-active-border
- status
- window-status
- window-status-current

Remember that the terminal is 256 colors, so using a hex color will make tmux approximate the closest avilable color. Eventually I preferred to pick colors by hand. I did consult [hexterm](https://www.npmjs.com/package/hexterm), and some other tools, but eventually resorted to this [256 color map](https://i.stack.imgur.com/e63et.png) and picked the colors that looked good to my eye.

{% gist 6957eab43957c83f56554ffdcced4100 vscode.tmux.conf %}

(put that in your .tmux.conf file)

## iTerm2

For iterm it's a plist xml. I created a very simple script to convert the hex colors to RGB values, and built the XML:

{% gist 6957eab43957c83f56554ffdcced4100 vscode.itermcolors %}

(load that from iTerm2 Preferences->Profiles->Colors->Color Presets...->Import...)


