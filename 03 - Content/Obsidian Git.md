---
aliases: 
tags:
  - 📝
primary categories:
  - "[[01 - Vault Administration]]"
secondary categories:
  - "[[02 - Obsidian]]"
type: Basic
---
# [[Obsidian Git]]
***
## Description:

Simple plugin that allows you to backup your [Obsidian.md](https://obsidian.md) vault to a remote git repository (e.g. private repo on GitHub). This plugin assumes you have existing git repository initialized locally and credentials are setup. This is the mechanism by which all your notes are sync'd to the Offpipe repository and shared between consultants. 

## Installation:

1. Obsidian Git is a registered Obsidian [Plugin]([[Obsidian - Plugins#Required]]) and and can be installed directly from `Settings > Community Plugins > Browse`

## Configuration

```ad-info
### Not Seeing this stylized in *PREVIEW* mode?
#### Try installing the community plugin, 'Admonition'

# Obsidian Git Settings:
1. *Vault Backup Interval*: 60
2. *Auto Pull Interval*: 10
3. *Commit message*: FLast {{date}}
4. *Date placeholder*: MM-DD-YYYY HH:mm:ss
5. *Pull changes before push*: Enabled (default)
```

## Commands

All associated commands specific to Obsidian git can be reviewed from the Command Pallete (**CTRL + P** to open)

![[Pasted image 20210907112215.png]]

These commands can be added to hotkey combinations which greatly simply the addons use. 

![[Pasted image 20210907112642.png]]

___

## Resources:

| Hyperlink                                                             | Info |
| --------------------------------------------------------------------- | ---- |
| [Obsidian Git Github Repo](https://github.com/denolehov/obsidian-git) |      | 

_Created Date_: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>
_Last Modified Date_: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
