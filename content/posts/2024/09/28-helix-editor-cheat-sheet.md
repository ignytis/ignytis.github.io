---
title: "Helix Editor: a cheat sheet"
date: 2024-09-28T16:23:11+02:00
description: The most used commands for Helix Editor
tags: ["helix-editor", "editors", "tools"]
draft: false
---

# About

Helix Editor is a terinal-based text editor writen in Rust.

https://helix-editor.com/


# The main article

https://docs.helix-editor.com/keymap.html

# General notes

Commands need to be typed in Normal Mode. To enterthe Normal Mode, press `ESC`

# Key bindings

## Navigation

### General

|                          	  |    |
|-----------------------------|----|
| Go to next word start	      | w  |
| Go to next WORD start	      | W  |
| Go to previous word start	  | b  |
| Go to previous WORD start	  | B  |
| Go to next word end	      | e  |
| Go to next WORD end	      | E  |
| Go to the first line        | gg |
| Go to the last line         | ge |

### Go to line with provided number

1. Press `g`
1. Type the line number
1. Press `g`

## Copy, paste

### Copy (yank) to system clipboard

1. Press `v`
1. Select your text using arrows or mouse pointer (if supported)
1. Press `SPACE`
1. Press `y`
1. Paste your text wherever your wish

## Search and replace

Unlike in some other editors, the Search and Replace operation in Helix is a chain of commands:

1. Press `%s` (do NOT add `:` before)
1. Type your search term and press `ENTER`
1. Press `c`
1. Now you have multiselect active with original values deleted. You can type the value to replace your original values with whatever you will type
1. Press `ENTER`
1. Press `,` to exit from multiselect mode

## Line operations

### Delete a line

1. Press `x`
1. Press `d`
