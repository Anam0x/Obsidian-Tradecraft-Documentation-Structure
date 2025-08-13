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
# [[Obsidian - Modifying Vault Structure]]
***
## Overview

This note covers instructions for how to add primary/secondary categories, content types, and their respective templates. In future releases of this project, I hope to add logic either internally to Obsidian or as external scripts to automate this behavior safely.

## Primary and Secondary Categories

The contents of this vault are divided into primary and secondary categories. Primary categories are the most basic categorization of all notes. Secondary categories are slightly more specific and can have multiple "parent" primary categories, effectively creating a vault with multiple overlapping tree topology structures with a depth of three. It is not advised to remove the role of primary and secondary categories from the vault, nor to add additional tertiary or additional categories.
### Adding New Primary and/or Secondary Categories

To add a new primary or secondary category, the user must simply create a new note in the `01 - Primary Categories` or `02 - Secondary Categories` directory and select the `🥇 (Primary Category)` or `🥈 (Secondary Category)` option, respectively. Obsidian will not accept empty note names and will automatically load the note structure associated with the selected category/content type.

#### TODO: Add GIF showing successful primary and secondary category creation

This behavior is implemented in the main logical section of the [[0400 - Gen_Note]] script.

```js
...
//////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////// Main //////////////////////////////////////
//////////////////////////////////////////////////////////////////////////////////

// Present selection of Search Tags to user
const noteTag = await tp.system.suggester(["🥇 (Primary Category)", "🥈 (Secondary Category)", "📝 (Basic)", "⛏️ (Tool)", "📕 (TTP)", "✅ (Playbook)", "🗺️ (Mindmap)", "⌛ (Debrief)", "💣 (Payload)", "🏗️ (Infrastructure)", "🎓 (Study Resources)"], ["Primary", "Secondary", "Basic", "Tool", "TTP", "Playbook", "Mindmap", "Debrief", "Payload", "Infrastructure", "Study Resources"], true);

// Ensure the user is not supplying an undefined or untitled note name
let title = tp.file.title;
title = await setTitle(title);

// Get the Struct associated with the Search Tag selected
const config = await getNoteStruct(noteTag);

// move file to appropriate corresponding folder
await tp.file.move(config.destination + "/" + config.prefix + title);

//build the note
const note = await buildNote(title,config);
```

Observe that if the user attempts to add a note type to a directory that does not match its destination, rather than throwing an error, Obsidian will create the note in the appropriate destination directory. For example, if a user attempts adding a `🥈 (Secondary Category)` note to the `01 - Primary Categories` directory with the name "`Test Category`", Obsidian will automatically create a note at `02 - Secondary Categories/02 - Test Category` using the template for secondary categories. This behavior extends to content notes as well; an attempt at adding a content note—such as `📝 (Basic)`, `⛏️ (Tool)`, or `📕 (TTP)`—to either category directory will result in a new file being created in the `03 - Content` directory with the appropriate content template enforced.

#### TODO: Add GIF showing expected behavior when adding a note type that does not belong in its destination directory

The templates for primary and secondary categories are deliberately minimal, changes can be made without breaking vault logic. There is currently no automation for retroactively updating previously created primary/secondary categories to reflect updates to the current template; this will either have to be done manually or automated in the future.

### Deleting Primary and/or Secondary Categories

Removing primary and/or secondary categories is as simple as right-clicking the category under it's respective directory and selecting *Delete*. This will not remove previous links to the category or notes that contain them; this will have to be corrected manually.

#### TODO: Add GIF showing deletion of example category, include link to said category

### Changing Category Hierarchy

Primary and secondary categories are critical elements of this vault structure, so it is unlikely that their role will ever be removed entirely from this vault. Adding subsequent category roles (e.g., tertiary, quaternary, etc.) risks adding excessive parent/child relationships to the vault—which is somewhat antithetical to the minimally hierarchical [[Zettlekasten]] technique that this vault is based on—and is therefore advised against.

If users of this vault still choose to pursue either or both of these scenarios however, the former can be achieved by removing references to the unwanted category type in [[0400 - Gen_Note]], deleting its `04 - Templates` sub-directory, and updating notes that link to primary/secondary categories manually. The latter can be achieved by creating a new directory to house new category types (ideally after `02 - Secondary Categories`, which would require incrementing all references to `03 - Content` and `04 - Templates`), updating [[0400 - Gen_Note]] to present a new category option and load new template structures, and finally creating said template structures for the new category.

In both cases, a modification will have to be made to the `isStructCategory` function of [[0400 - Gen_Note]] to return the correct boolean value. While building a note, Obsidian determines whether a note is a category by checking the `name` property of the `noteStructConfig` object; if a note is a category, the header template structure element is omitted while building the note, otherwise it is included.

```js
...
// Name: isStructCategory
// Description: Determines whether or not the struct is related to a category
// by its name equaling "Primary" or "Secondary"
// Returns: Boolean
async function isStructCategory(structName) {
	// dont build out template elements, just reference
	if (structName === "Primary" || structName === "Secondary") {
		return true;
	} 
	return false;
}

// Name: buildNote
// Description: Performs the actual concatenation of our discrete header, body and 
// footer note elements then handles the creation of the note itself.
// Returns: String
async function buildNote(title,noteStructConfig) {
	
	// Notes:
	// - Leaving the search tag blank for now
	// - Headers are intentionally left mostly blank for now
	// - Timestamp has to be manually added as literal string; attempting to add timestamp
	// as a property or included file results in "NaN" rendering or other errors
	// - Removed setMetdata function; setting properties in included files is easier IMO
	// - Removed shouldLink function; purpose unclear, seems unnecessary
	const meta = await tp.file.include(noteStructConfig.metadata);
	let prefix = noteStructConfig.prefix;
	const pageTitle = "# [[" + prefix + title + "]]\n";
	const body = await tp.file.include(noteStructConfig.body);
	const time = await setTimestamps();
	const isCategory = await isStructCategory(noteStructConfig.name);
	
	if (isCategory === true) {
		return meta + pageTitle + body + time;
	}
	else if (isCategory === false) {

		const head = await tp.file.include(noteStructConfig.header);
		return meta + pageTitle + head + body + time;
	}
}
...
```

## Content

Content notes represent the lowest depth of the vault structure as individual "ideas". I have created multiple content "types" to anticipate the likeliest structure these notes will take (e.g., "`⛏️ (Tool)`", "`📕 (TTP)`", "`✅ (Playbook)`", and etc.) It is likely that new content types will have to be created and old ones removed as the vault conforms to the user's personal learning and assessment methodology.
### Adding New Content Types

Adding a new content type currently requires manual changes to the [[0400 - Gen_Note]] script and adding template structures to the `04 - Templates/04 - Content` sub-directory:
1. Modify the `noteTag` declaration in [[0400 - Gen_Note]] to include the new content type
2. Modify the `getNoteStruct` function in [[0400 - Gen_Note]] to load new template structures
3. Create sub-directory in `04 - Templates/04 - Content` for new content type template elements and appropriately prefixed files for the content type's metadata, header, and body

In this example, we want to add a new content type called "`Content Example`" which is associated with the emoji 😘. We will modify the `noteTag` declaration statement to include `😘 (Content Example)` as a valid option and map it to the value `Content Example`. The order of keys and values is critical, otherwise some options the user selects will be mapped to incorrect template structures.

```js
...
//////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////// Main //////////////////////////////////////
//////////////////////////////////////////////////////////////////////////////////

// Present selection of Search Tags to user
const noteTag = await tp.system.suggester(["🥇 (Primary Category)", "🥈 (Secondary Category)", "📝 (Basic)", "⛏️ (Tool)", "📕 (TTP)", "✅ (Playbook)", "🗺️ (Mindmap)", "⌛ (Debrief)", "💣 (Payload)", "🏗️ (Infrastructure)", "🎓 (Study Resources)", "😘 (Content Example)"], ["Primary", "Secondary", "Basic", "Tool", "TTP", "Playbook", "Mindmap", "Debrief", "Payload", "Infrastructure", "Study Resources", "Content Example"], true);
...
```

Next, we need to modify the script logic to load a new template structure if the user chooses this note. Scroll up to the `getNoteStruct` function in [[0400 - Gen_Note]] and add a new `else if` case covering the new content type. If Obsidian does not detect a valid entry, it will prompt again for the user to select a valid Search Tag; this declaration will also have to be updated to match the original declaration in the main logic of the script.

> *Note*:
> We have not created the `04100X - XXXXX` notes referenced in the `noteStructConfig` dictionary yet. We will create these templates in the next step. Make sure that the prefixes' ID is incremented correctly; for example, if the previous template notes have the prefixes `040901`, `040902`, and `040903`, the notes referenced in the `metadata`, `header`, and `body` properties of the returned `noteStructConfig` dictionary will have the prefixes `041001`, `041002`, `041003`, respectively.

```js
// Name: getNoteStruct
// Description: Accepts the Search Tag (emoji) selected by the user
// and returns the appropriate note structure configuration details
// Return: Dictionary
async function getNoteStruct(noteTag) {
	console.log("Getting Struct MD for tag: " + noteTag);
	var noteStructConfig;
	
	if (noteTag.startsWith("Primary")) {
		noteStructConfig = {
			name: "Primary",
			prefix: "01 - ",
			destination: "01 - Primary Categories/",
			metadata: "[[04 - Templates/04 - Primary Category/0401 - Metadata]]",
			body: "[[04 - Templates/04 - Primary Category/0402 - Body]]"
		};
	...
	} else if (noteTag.startsWith("Study Resources")) {
		noteStructConfig = {
			name: "Study Resources",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040901 - Metadata]]",
			header: "[[040902 - Header]]",
			body: "[[040903 - Body]]"
		};
	} else if (noteTag.startsWith("Content Example")) {
		noteStructConfig = {
			name: "Content Example",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[041001 - Metadata]]",
			header: "[[041002 - Header]]",
			body: "[[041003 - Body]]"
		};
	} else {
		console.log("You selected an option outside what was expected.");
		console.log("Try again.");
		const newNoteTag = await tp.system.suggester(["🥇 (Primary Category)", "🥈 (Secondary Category)", "📝 (Basic)", "⛏️ (Tool)", "📕 (TTP)", "✅ (Playbook)", "🗺️ (Mindmap)", "⌛ (Debrief)", "💣 (Payload)", "🏗️ (Infrastructure)", "🎓 (Study Resources)", "😘 (Content Example)"], ["Primary", "Secondary", "Basic", "Tool", "TTP", "Playbook", "Mindmap", "Debrief", "Payload", "Infrastructure", "Study Resources", "Content Example"], true);
		return await getNoteStruct(newNoteTag);
	}
	return noteStructConfig;
}
...
```

The final step is to create the respective templates for the new content type. It is advised to create a copy of a template structure that best matches the desired new template structure and make any necessary changes to the duplicate. It's critical to ensure that the name of the metadata, header, and body files matches the expected value in the `getNoteStruct` function of [[0400 - Gen_Note]], otherwise these structures will never be imported.

#### TODO: Add GIF demonstrating how to create new content type's template elements

The new content type should be ready to use now. Note how the new content inherits the template structural elements we previously created and is correctly routed to the `03 - Content` directory.

#### TODO: Add GIF demonstrating correct use of new content type

The instructions for deleting a content type is the inverse of that of creating a new content type, with the addition of removing notes that use that content type:
1. Remove the reference to the undesired content type in the `noteTag` declaration in [[0400 - Gen_Note]]
2. Remove the reference to the undesired content type and its template structures in the `getNoteStruct` function in [[0400 - Gen_Note]]
3. Delete the sub-directory for the undesired content type in `04 - Templates/04 - Content/`; it is advised to decrement the prefixes for any content types affected by this deletion and ensure that those changes are accounted for in the `getNoteStruct` function in [[0400 - Gen_Note]]
4. Delete all instances of content notes that were previously instantiated using this type or modify its metadata properties and body to implement a new content type

Removing instances of a previously deleted content type is advised because if ignored the vault structure will contain multiple content notes with inaccurate `tags` and `type` fields in its metadata section. This can frustrate searching efforts or produce confusing results in Obsidian's graph view.
## Templates

There are currently three elements of a given note type: 1) metadata, 2) header, and 3) body. Primary and secondary categories only have metadata and body elements, while content types have all three. Modifying template structures can be done without adversely impacting the vault, but adding or removing template structures or entire templates will require handling with care.

### Adding or Removing Template Structural Elements

In the future, it may become appropriate or necessary to add additional template elements or remove them altogether. In either case, the entire `getNoteStruct` function in [[0400 - Gen_Note]] will have to be modified so that every category or content type that uses the new template structure or currently uses a deleted template structure reflects this change.

Additionally, the `buildNote` function in [[0400 - Gen_Note]] will have to be modified. Currently, there is no logic to anticipate cases where the `metadata`, `prefix`, `body`, or `name` properties are not defined in a `noteStructConfig` object; the `metadata` and `body` properties represent the template structures for metadata and body elements, respectively. The `header` property, which represents the header template structure element, is only loaded if the note is not a category. Depending on how the vault user wants notes to utilize template structure elements, this function will have to be modified to anticipate the desired note structure while not breaking the construction of other note types.

> *Note*:
> It might be a good idea for future releases of this vault to build notes more dynamically as opposed to expecting `noteStructConfig` objects to always contain a minimum set of properties.

```js
...
// Name: getNoteStruct
// Description: Accepts the Search Tag (emoji) selected by the user
// and returns the appropriate note structure configuration details
// Return: Dictionary
async function getNoteStruct(noteTag) {
	console.log("Getting Struct MD for tag: " + noteTag);
	var noteStructConfig;
	
	if (noteTag.startsWith("Primary")) {
		noteStructConfig = {
			name: "Primary",
			prefix: "01 - ",
			destination: "01 - Primary Categories/",
			metadata: "[[04 - Templates/04 - Primary Category/0401 - Metadata]]",
			body: "[[04 - Templates/04 - Primary Category/0402 - Body]]"
		};
	...
	} else {
		console.log("You selected an option outside what was expected.");
		console.log("Try again.");
		const newNoteTag = await tp.system.suggester(["🥇 (Primary Category)", "🥈 (Secondary Category)", "📝 (Basic)", "⛏️ (Tool)", "📕 (TTP)", "✅ (Playbook)", "🗺️ (Mindmap)", "⌛ (Debrief)", "💣 (Payload)", "🏗️ (Infrastructure)", "🎓 (Study Resources)"], ["Primary", "Secondary", "Basic", "Tool", "TTP", "Playbook", "Mindmap", "Debrief", "Payload", "Infrastructure", "Study Resources"], true);
		return await getNoteStruct(newNoteTag);
	}
	return noteStructConfig;
}

...

// Name: buildNote
// Description: Performs the actual concatenation of our discrete header, body and 
// footer note elements then handles the creation of the note itself.
// Returns: String
async function buildNote(title,noteStructConfig) {
	
	// Notes:
	// - Leaving the search tag blank for now
	// - Headers are intentionally left mostly blank for now
	// - Timestamp has to be manually added as literal string; attempting to add timestamp
	// as a property or included file results in "NaN" rendering or other errors
	// - Removed setMetdata function; setting properties in included files is easier IMO
	// - Removed shouldLink function; purpose unclear, seems unnecessary
	const meta = await tp.file.include(noteStructConfig.metadata);
	let prefix = noteStructConfig.prefix;
	const pageTitle = "# [[" + prefix + title + "]]\n";
	const body = await tp.file.include(noteStructConfig.body);
	const time = await setTimestamps();
	const isCategory = await isStructCategory(noteStructConfig.name);
	
	if (isCategory === true) {
		return meta + pageTitle + body + time;
	}
	else if (isCategory === false) {

		const head = await tp.file.include(noteStructConfig.header);
		return meta + pageTitle + head + body + time;
	}
}
...
```

### Adding or Removing Entire Templates

It's not advised to remove entire template sub-directories unless you are also deleting the note type associated with said template. If this is the desired effect however, both the `getNoteStruct` and `buildNote` functions of [[0400 - Gen_Note]] will have to be modified to anticipate note types not loading template structures at all. In this example, we have deleted the `04 - Templates/04 - Primary Category` sub-directory and modified the [[0400 - Gen_Note]] script to not load a template for notes created with the `🥇 (Primary Category)` option. Templates will still be enforced for secondary categories and all content types.

```js
...
// Name: getNoteStruct
// Description: Accepts the Search Tag (emoji) selected by the user
// and returns the appropriate note structure configuration details
// Return: Dictionary
async function getNoteStruct(noteTag) {
	console.log("Getting Struct MD for tag: " + noteTag);
	var noteStructConfig;
	
	if (noteTag.startsWith("Primary")) {
		noteStructConfig = {
			name: "Primary",
			prefix: "01 - ",
			destination: "01 - Primary Categories/"
		};
	...
	} else {
		console.log("You selected an option outside what was expected.");
		console.log("Try again.");
		const newNoteTag = await tp.system.suggester(["🥇 (Primary Category)", "🥈 (Secondary Category)", "📝 (Basic)", "⛏️ (Tool)", "📕 (TTP)", "✅ (Playbook)", "🗺️ (Mindmap)", "⌛ (Debrief)", "💣 (Payload)", "🏗️ (Infrastructure)", "🎓 (Study Resources)"], ["Primary", "Secondary", "Basic", "Tool", "TTP", "Playbook", "Mindmap", "Debrief", "Payload", "Infrastructure", "Study Resources"], true);
		return await getNoteStruct(newNoteTag);
	}
	return noteStructConfig;
}

// Name: buildNote
// Description: Performs the actual concatenation of our discrete header, body and 
// footer note elements then handles the creation of the note itself.
// Returns: String
async function buildNote(title,noteStructConfig) {
	
	// Notes:
	// - Leaving the search tag blank for now
	// - Headers are intentionally left mostly blank for now
	// - Timestamp has to be manually added as literal string; attempting to add timestamp
	// as a property or included file results in "NaN" rendering or other errors
	// - Removed setMetdata function; setting properties in included files is easier IMO
	// - Removed shouldLink function; purpose unclear, seems unnecessary
	// - If file is a "Primary" note, do not attempt loading the template structure fields
	const name = await tp.file.include(noteStructConfig.name);

	let prefix = noteStructConfig.prefix;
	const pageTitle = "# [[" + prefix + title + "]]\n";
	const time = await setTimestamps();
	const isCategory = await isStructCategory(name);
	
	if (name === "Primary") {
		return pageTitle + body + time;
	}
	else {
		const meta = await tp.file.include(noteStructConfig.metadata);
		const body = await tp.file.include(noteStructConfig.body);

		if (isCategory === true) {
			return meta + pageTitle + body + time;
		} else if {

			const head = await tp.file.include(noteStructConfig.header);
			return meta + pageTitle + head + body + time;
	}
	}
}
```

***
## Resources:

| Hyperlink | Info |
| --------- | ---- |
|           |      |

[^1]: 

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
