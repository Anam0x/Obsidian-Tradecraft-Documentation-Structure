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
# [[Dataview]]
***
## Description:

Creates dynamic views, tables, lists, and calendars by querying your notes' metadata, tags, links, and content.

## Installation:

1. Dataview is a registered Obsidian plugin and can be installed directly from `Settings > Community Plugins > Browse`
	* [[Obsidian - Plugins#Additional]]

## Configuration

* Enable JavaScript queries if you need advanced scripting capabilities (disabled by default for security) by navigating to `Settings > Community Plugins > Dataview` and toggling *Enable JavaScript queries*
* Set inline query prefix (default: =) for inline Dataview expressions by navigating to `Settings > Community Plugins > Dataview > Codeblocks` and setting the *Inline query prefix* value
* Enable/disable automatic task completion date tracking by navigating to `Settings > Community Plugins > Dataview > Tasks` and toggling *Automatic task completion tracking*

## Basic Usage

### Query Types

* **TABLE**: Display data in tabular format
* **LIST**: Show results as bulleted lists
* **TASK**: Display tasks with completion status
* **CALENDAR**: Show dates in calendar view

### Example Queries

List all notes tagged with #🥇 :
```dataview
LIST
FROM #🥇
```

Table of primary categories with dates:
```dataview
TABLE file.ctime as "Created", file.mtime as "Modified"
FROM "01 - Primary Categories"
SORT file.ctime DESC
```

Tasks due this week:
```dataview
TASK
WHERE due >= date(today) AND due <= date(today) + dur(1 week)
```

***
## Resources:

| Hyperlink                                                                                       | Info                   |
| ----------------------------------------------------------------------------------------------- | ---------------------- |
| [Dataview GitHub Repository](https://github.com/blacksmithgu/obsidian-dataview)                 | Main repository        |
| [Dataview Documentation](https://blacksmithgu.github.io/obsidian-dataview/)                     | Official documentation |
| [Query Language Reference](https://blacksmithgu.github.io/obsidian-dataview/queries/structure/) | Dataview syntax guide  |

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
