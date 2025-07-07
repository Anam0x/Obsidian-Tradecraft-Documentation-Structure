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
# [[Templater]]
***

## Description:

[Templater](https://github.com/SilentVoid13/Templater) is a template language that lets you insert **variables** and **functions** results into your [Obsidian](https://obsidian.md/) notes. 

![templater_demo](https://github.com/SilentVoid13/Templater/blob/561ac7bb30dbc2aff6aeab0dd7aa9883bef4fca8/imgs/templater_demo.gif?raw=true)

## How we'll use Templater
- Prompt for the Search tag on note creation
- Prompt for title on note creation
- Add date *created* automatically
- Update *last updated* automatically
- Move the note to the appropriate location
 	- 02 - Secondary Categories
	- 03 - Content
	- 05 - Personal

## Installation:

1. Templater is a registered Obsidian plugin and can be installed directly from `Settings > Community Plugins > Browse`
	* [[Obsidian - Plugins#Required]]

## Configuration

```ad-info
### Not Seeing this stylized in *PREVIEW* mode?
#### Try installing the community plugin, 'Admonition'

1. *Template Folder Location*: 04 - Templates
2. *Trigger Templater on New File Creation*: True
3. *Empty File Template*: 04 - Templates/0400 - Gen_Note
```

To help manage **incomplete**, **NULL**, or **'Untitled'** notes, it can  be helpful to assign `Settings > Files & Links > Folder to Create New Notes in` to `.trash`. Since our **0400 - Gen_Note** template handles moving successfully created notes to their appropriate folders, the categories listed above will automatically end up in trash. 

___

# Resources:

| Hyperlink                                                                 | Info |
| ------------------------------------------------------------------------- | ---- |
| [Templater Github Repo](https://github.com/SilentVoid13/Templater)        |      |
| [Templater Documentation](https://silentvoid13.github.io/Templater/docs/) |      |

[^1]: 

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
