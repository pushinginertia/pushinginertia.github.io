---
layout: post
title: Sublime Text Key Mappings for MacOS
tags: [Configuration]
---

This is my Sublime Text custom key configuration on MacOS, which makes it a bit more consistent with other text editors.

Open Preferences > Key Bindings – User and paste the following.

```json
[
    // Reference:
    // http://docs.sublimetext.info/en/sublime-text-2/reference/commands.html
    {"keys": ["end"], "command": "move_to", "args": {"to": "eol"}},
    {"keys": ["home"], "command": "move_to", "args": {"to": "bol"}},
    {"keys": ["super+home"], "command": "move_to", "args": {"to": "bof"}},
    {"keys": ["super+end"], "command": "move_to", "args": {"to": "eof"}},

    { "keys": ["alt+left"], "command": "move", "args": {"by": "subwords", "forward": false}},
    { "keys": ["alt+right"], "command": "move", "args": {"by": "subword_ends", "forward": true}},
    { "keys": ["alt+shift+left"], "command": "move", "args": {"by": "subwords", "forward": false, "extend": true}},
    { "keys": ["alt+shift+right"], "command": "move", "args": {"by": "subword_ends", "forward": true, "extend": true}},

    { "keys": ["ctrl+left"], "command": "prev_view" },
    { "keys": ["ctrl+right"], "command": "next_view" },

    {"keys": ["super+g"], "command": "show_overlay", "args": {"overlay": "goto", "text": ":"}},
    {"keys": ["super+d"], "command": "duplicate_line"},
    {"keys": ["super+r"], "command": "show_panel", "args": {"panel": "replace"}},
]
```
