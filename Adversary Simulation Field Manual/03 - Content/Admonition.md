---
aliases:
tags:
  - 📝Basic
primary categories:
  - "[[Vault Administration]]"
secondary categories:
  - "[[Obsidian]]"
type: Basic
---
# [[Admonition]]

***

## Description

Adds admonition block-styled content to Obsidian.md, styled after [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/reference/admonitions/)

![](https://raw.githubusercontent.com/javalent/admonitions/master/publish/gifs/all.gif)

## Installation

1. Admonition is a registered Obsidian plugin and can be installed directly from `Settings > Community Plugins > Browse`
	* [[Obsidian - Plugins#Additional]]

## Configuration

* Nothing required beyond installation for default admonition block element display
* Would recommend enabling the 'copy' button permitting the copying of content directly from the blocks

## Basic Usage

Use the following formats for different types of admonitions:

### Code Block Style

````markdown
```ad-<type>
title: <Optional Title>
collapse: <none|open|close>
icon: <FontAwesome or RPG Awesome icon name>
color: <R,G,B>
Your content here.
```
````

```ad-info
title: Custom Title
collapse: none
Your content here.
```

#### Supported Parameters

| Parameter   | Description                                                          |
| ----------- | -------------------------------------------------------------------- |
| `title:`    | Optional. Overrides default title; supports Markdown in block-style. |
| `collapse:` | `open`, `close`, or `none`. Controls collapsible behavior.           |
| `icon:`     | Sets a custom icon using FontAwesome or RPG Awesome names.           |
| `color:`    | RGB triad (e.g., `200,200,200`) to override default styling.         |

#### Supported Types

| Type       | Aliases                           | Use Case                                                       |
| ---------- | --------------------------------- | -------------------------------------------------------------- |
| `note`     | `note`, `seealso`                 | General notes, references, or related information.             |
| `abstract` | `abstract`, `summary`, `tldr`     | Summarizing content or providing an overview of a section.     |
| `info`     | `info`, `todo`                    | Highlighting helpful information or listing tasks to complete. |
| `tip`      | `tip`, `hint`, `important`        | Sharing helpful tips, best practices, or important details.    |
| `success`  | `success`, `check`, `done`        | Indicating completion, success, or confirmation messages.      |
| `question` | `question`, `help`, `faq`         | Posing questions, FAQs, or prompts for further thinking.       |
| `warning`  | `warning`, `caution`, `attention` | Calling out cautions, risks, or important alerts.              |
| `failure`  | `failure`, `fail`, `missing`      | Noting missing items, failed tasks, or critical issues.        |
| `danger`   | `danger`, `error`                 | Highlighting severe problems, errors, or urgent issues.        |
| `bug`      | `bug`                             | Tracking bugs, issues, or debugging notes.                     |
| `example`  | `example`                         | Providing code examples, demonstrations, or sample content.    |
| `quote`    | `quote`, `cite`                   | Highlighting quotes, citations, or references.                 |

### Nesting

For block-style admonitions, wrap with matching backtick levels.

`````markdown
````ad-note
title: Parent
```ad-warning
title: Nested Child
Nested content.
```
Parent content
````
`````

````ad-note
title: Parent
```ad-warning
title: Nested Child
Nested content.
```
Parent content
````

___

## Resources

| Hyperlink                                                                           | Info            |
| ----------------------------------------------------------------------------------- | --------------- |
| [Admonition GitHub Repository](https://github.com/valentine195/obsidian-admonition) | Main repository |

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
