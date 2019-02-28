---
layout: post
title: Configuring Eclipse for IntelliJ users
tags: [Configuration]
---

There are a few things I've gotten used to in IntelliJ that require some customizations in Eclipse. The following steps will make Eclipse feel more familiar and ease the transition.

## Key Bindings

You can get close to IntelliJ's key bindings by installing the ideakeyscheme plugin.

Place the downloaded JAR in eclipse/dropins and unzip it into a subdirectory. Open Eclipse > Preferences > General > Keys and select the ‘IntelliJ Idea' scheme.

The close window shortcut (Cmd + W) won't work because it's bound to the ‘Select Enclosing Element' command. Manually remove the key unbinding to fix this.

Unbind the following:

* Toggle Split Editor (Vertical): Cmd + Shift + [

Additional key bindings:

* Next Tab: Ctrl + → and Cmd + Shift + [
* Previous Tab: Ctrl + ← and Cmd + Shift + ]
* Line Start: fn + ←
* Line End: fn + →
* References in Workspace: Alt + F7
* Open Type Hierarchy: Ctrl + H

## Copy/Cut/Paste

Install the LineCopyPaste plugin to mimick the behaviour of IntelliJ. For example, pressing Cmd + X when no text is selected will cut the entire line instead of doing nothing.

## Color Theme

[Grab it here.](http://eclipsecolorthemes.org/?view=theme&id=935)

## Show print margin

Select Preferences > General > Editors > Text Editors > Show print margin and set the column size accordingly.

## Disable automatic highlighting of usages

Preferences > Java > Editor > Mark Occurrences

I'm personally not a fan of things flickering on and off as I move my cursor around the screen, but Eclipse's behaviour to highlight usages (equivalent of IntelliJ's Cmd + Shift + F7) is very rudimentary and takes focus away from the edit window, requiring a mouse click to get back in.

## Support .py files

Eclipse defaults to starting an external editor for Python scripts (Xcode on MacOS). Fix this by installing the PyDev plugin.

* Help > Install New Software
* Add `http://pydev.sf.net/updates/` as a new site.
* Follow on-screen instructions to install PyDev.
* Restart Eclipse.

## Support .sh files

Unfortunately, Eclipse supports very little on a fresh installation and requires a plugin to support pretty much anything other than a .java or .xml file. Install ShellEd for shell script support.

This has a dependency that won't exist on a fresh Eclipse installation, so first go to _Install Software_ and install WST Server Adaptors, then restart Eclipse.

Use Eclipse's Install Software interface by adding the software site: `http://sourceforge.net/projects/shelled/update/`

Install ShellEd through the interface.

## Autosave changes

Eclipse has some configuration options to automatically save changes when a build is triggered, but requires a plugin to save changes on a timed basis and when a file loses focus. Smartsave for Eclipse should do the trick.

