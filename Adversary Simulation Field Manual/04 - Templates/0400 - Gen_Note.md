<%* 
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////// Functions /////////////////////////////////////
//////////////////////////////////////////////////////////////////////////////////

// Name: setTitle
// Description: Ensure a valid title is supplied, or prompt user for title.
// Rename the created file from "Untitled" to what they've supplied.
async function setTitle(title) {
	if (await isValidTitle(title)) {
		console.log(`Title: "${title}" is valid`);
		await tp.file.rename(title);
		return title;
	} else {
		const newTitle = await tp.system.prompt("Title of New Note");
		return await setTitle(newTitle);
	}
}

// Name: setTag
// Description: Ensure a valid search tag is selected, keep prompting until one is chosen
async function setTag() {
	let selectedTag = null;
	while (!selectedTag) {
		selectedTag = await tp.system.suggester(
			["🥇 (Primary Category)", "🥈 (Secondary Category)", "📝 (Basic)", "⛏️ (Tool)", "📕 (TTP)", "✅ (Playbook)", "🗺️ (Mindmap)", "⌛ (Debrief)", "💣 (Payload)", "🏗️ (Infrastructure)", "🎓 (Study Resources)"], 
			["Primary", "Secondary", "Basic", "Tool", "TTP", "Playbook", "Mindmap", "Debrief", "Payload", "Infrastructure", "Study Resources"], 
			false,
			"Select note type (required):"
		);
		// If null/undefined, loop continues
	}
	return selectedTag;
}

// Name: setCategories
// Description: Ensure valid categories are selected, keep prompting until at least one is chosen
async function setCategories(categoryType) {
	let selectedCategories = [];
	while (selectedCategories.length === 0) {
		selectedCategories = await selectCategories(categoryType);
		// If empty array or null, loop continues
	}
	return selectedCategories;
}

// Name: isValidTitle
// Description: Checks supplied string (title) against 'undefined' type , null 
// value, empty value and when the type is a string - we validate if it follows 
// the default naming scheme of 'Untitled #' - rejects assignment of any of these 
// cases
// Return: Boolean
async function isValidTitle(title) {
	// if nothing was supplied, try again
	if (typeof title === 'undefined' || (typeof title === 'string' && title.includes('Untitled')) || title === null || title === "") 
	{
		return false;
	} 
	else 
	{
		return true;
	}
}

// Name: getPrimaryCategories
// Description: Gets list of all primary category files in the vault
// Return: Array of primary category names (without prefixes)
async function getPrimaryCategories() {
	const primaryCatFolder = app.vault.getAbstractFileByPath("01 - Primary Categories");
	if (!primaryCatFolder) {
		return [];
	}
	
	const files = primaryCatFolder.children
		.filter(file => file.extension === "md")
		.map(file => file.basename.replace(/^01 - /, "")); // Remove "01 - " prefix
	
	return files;
}

// Name: getSecondaryCategories
// Description: Gets list of all secondary category files in the vault
// Return: Array of secondary category names (without prefixes)
async function getSecondaryCategories() {
	const secondaryCatFolder = app.vault.getAbstractFileByPath("02 - Secondary Categories");
	if (!secondaryCatFolder) {
		return [];
	}
	
	const files = secondaryCatFolder.children
		.filter(file => file.extension === "md")
		.map(file => file.basename.replace(/^02 - /, "")); // Remove "02 - " prefix
	
	return files;
}

// Name: selectCategories
// Description: Prompts user to select one or more categories (primary or secondary)
// Return: Array of selected category names with proper linking format
async function selectCategories(categoryType) {
	const isSecondary = categoryType === "secondary";
	const availableCategories = isSecondary ? await getSecondaryCategories() : await getPrimaryCategories();
	const prefix = isSecondary ? "02 - " : "01 - ";
	const categoryLabel = isSecondary ? "SECONDARY" : "PRIMARY";
	
	if (availableCategories.length === 0) {
		console.log(`No ${categoryType} categories found`);
		return [];
	}
	
	const selectedCategories = [];
	let continueSelecting = true;
	
	while (continueSelecting && selectedCategories.length < availableCategories.length) {
		// Filter out already selected categories
		const remainingCategories = availableCategories.filter(cat => !selectedCategories.includes(cat));
		
		if (remainingCategories.length === 0) {
			break;
		}
		
		// Add "Done" option if at least one category is selected
		const options = [...remainingCategories];
		const displayOptions = [...remainingCategories];
		
		if (selectedCategories.length > 0) {
			options.push("Done");
			displayOptions.push("✅ Done (Finish Selection)");
		}
		
		const promptText = selectedCategories.length === 0 
			? `Select ${categoryLabel} category to link back to:` 
			: `Selected: ${selectedCategories.join(", ")}. Select another or choose Done:`;
		
		const selection = await tp.system.suggester(displayOptions, options, false, promptText);
		
		if (selection === "Done" || !selection) {
			continueSelecting = false;
		} else {
			selectedCategories.push(selection);
		}
	}
	
	// Format as proper wiki links with appropriate prefix
	return selectedCategories.map(cat => `"[[${prefix}${cat}]]"`);
}

// Name: getNoteStruct
// Description: Accepts the Search Tag (emoji) selected by the user
// and returns the appropriate note structure configuration details
// Return: Dictionary
async function getNoteStruct(noteTag, primaryCategories = [], secondaryCategories = []) {
	console.log("Getting Struct MD for tag: " + noteTag);
	var noteStructConfig;
	
	if (noteTag.startsWith("Primary")) {
		noteStructConfig = {
			name: "Primary",
			prefix: "01 - ",
			destination: "01 - Primary Categories/",
			metadata: "[[04 - Templates/04 - Primary Category/0401 - Metadata]]",
			body: "[[04 - Templates/04 - Primary Category/0402 - Body]]",
			primaryCategories: [],
			secondaryCategories: []
		};
	} else if (noteTag.startsWith("Secondary")) {
		noteStructConfig = {
			name: "Secondary",
			prefix: "02 - ",
			destination: "02 - Secondary Categories/",
			metadata: "[[04 - Templates/04 - Secondary Category/0401 - Metadata]]",
			body: "[[04 - Templates/04 - Secondary Category/0402 - Body]]",
			primaryCategories: primaryCategories,
			secondaryCategories: []
		};
	} else if (noteTag.startsWith("Basic")) {
		noteStructConfig = {
			name: "Basic",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040101 - Metadata]]",
			header: "[[040102 - Header]]",
			body: "[[040103 - Body]]",
			primaryCategories: primaryCategories,
			secondaryCategories: secondaryCategories
		};
	} else if (noteTag.startsWith("Tool")) {
		noteStructConfig = {
			name: "Tool",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040201 - Metadata]]",
			header: "[[040202 - Header]]",
			body: "[[040203 - Body]]",
			primaryCategories: primaryCategories,
			secondaryCategories: secondaryCategories
		};
	} else if (noteTag.startsWith("TTP")) {
		noteStructConfig = {
			name: "TTP",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040301 - Metadata]]",
			header: "[[040302 - Header]]",
			body: "[[040303 - Body]]",
			primaryCategories: primaryCategories,
			secondaryCategories: secondaryCategories
		};
	} else if (noteTag.startsWith("Playbook")) {
		noteStructConfig = {
			name: "Playbook",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040401 - Metadata]]",
			header: "[[040402 - Header]]",
			body: "[[040403 - Body]]",
			primaryCategories: primaryCategories,
			secondaryCategories: secondaryCategories
		};
	} else if (noteTag.startsWith("Mindmap")) {
		noteStructConfig = {
			name: "Mindmap",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040501 - Metadata]]",
			header: "[[040502 - Header]]",
			body: "[[040503 - Body]]",
			primaryCategories: primaryCategories,
			secondaryCategories: secondaryCategories
		};
	} else if (noteTag.startsWith("Debrief")) {
		noteStructConfig = {
			name: "Debrief",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040601 - Metadata]]",
			header: "[[040602 - Header]]",
			body: "[[040603 - Body]]",
			primaryCategories: primaryCategories,
			secondaryCategories: secondaryCategories
		};
	} else if (noteTag.startsWith("Payload")) {
		noteStructConfig = {
			name: "Payload",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040701 - Metadata]]",
			header: "[[040702 - Header]]",
			body: "[[040703 - Body]]",
			primaryCategories: primaryCategories,
			secondaryCategories: secondaryCategories
		};
	} else if (noteTag.startsWith("Infrastructure")) {
		noteStructConfig = {
			name: "Infrastructure",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040801 - Metadata]]",
			header: "[[040802 - Header]]",
			body: "[[040803 - Body]]",
			primaryCategories: primaryCategories,
			secondaryCategories: secondaryCategories
		};
	} else if (noteTag.startsWith("Study Resources")) {
		noteStructConfig = {
			name: "Study Resources",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040901 - Metadata]]",
			header: "[[040902 - Header]]",
			body: "[[040903 - Body]]",
			primaryCategories: primaryCategories,
			secondaryCategories: secondaryCategories
		};
	} else {
		console.log("You selected an option outside what was expected.");
		console.log("Try again.");
		const newNoteTag = await tp.system.suggester(["🥇 (Primary Category)", "🥈 (Secondary Category)", "📝 (Basic)", "⛏️ (Tool)", "📕 (TTP)", "✅ (Playbook)", "🗺️ (Mindmap)", "⌛ (Debrief)", "💣 (Payload)", "🏗️ (Infrastructure)", "🎓 (Study Resources)"], ["Primary", "Secondary", "Basic", "Tool", "TTP", "Playbook", "Mindmap", "Debrief", "Payload", "Infrastructure", "Study Resources"], true);
		return await getNoteStruct(newNoteTag);
	}
	return noteStructConfig;
}

// Name: setTimestamps
// Description: Builds the timestamps element appended to note pages as they're built. Includes the definition of the Modified Date dynamic element which is updated when the page is edited and shown at the time of the page being rendered in preview mode.
async function setTimestamps() {
	return "\n*Created Date*: <%+tp.file.creation_date(\"MMMM Do YYYY (HH:mm a)\")%\>  \n*Last Modified Date*: \<%+tp.file.last_modified_date(\"MMMM Do YYYY (HH:mm a)\")%\>";
}

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
	// - Headers are intentionally left mostly blank for now
	// - Timestamp has to be manually added as literal string; attempting to add timestamp
	// as a property or included file results in "NaN" rendering or other errors
	// - Removed setMetdata function; setting properties in included files is easier IMO
	// - Removed shouldLink function; purpose unclear, seems unnecessary
	let meta = await tp.file.include(noteStructConfig.metadata);
	
	// If this is a secondary category with selected primary categories, update the metadata
	if (noteStructConfig.name === "Secondary" && noteStructConfig.primaryCategories.length > 0) {
		const primaryCatList = noteStructConfig.primaryCategories.join('\n  - ');
		meta = meta.replace(
			'primary categories:\n  - Add link(s) [[]] back to related PRIMARY categories',
			`primary categories:\n  - ${primaryCatList}`
		);
	}

	// If this is content note with selected categories, update the metadata
	const isContentNote = !["Primary", "Secondary"].includes(noteStructConfig.name);
	if (isContentNote) {
		// Update primary categories if any selected
		if (noteStructConfig.primaryCategories.length > 0) {
			const primaryCatList = noteStructConfig.primaryCategories.join('\n  - ');
			meta = meta.replace(
				'primary categories:\n  - Add link(s) [[]] back to related PRIMARY categories',
				`primary categories:\n  - ${primaryCatList}`
			);
		}

		// Update secondary categories if any selected
		if (noteStructConfig.secondaryCategories.length > 0) {
			const secondaryCatList = noteStructConfig.secondaryCategories.join('\n  - ');
			meta = meta.replace(
				'secondary categories:\n  - Add link(s) [[]] back to related SECONDARY categories',
				`secondary categories:\n  - ${secondaryCatList}`
			);
		}
	}
	
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

// Name: isContentNote
// Description: Determines if the note type is a content note that should prompt for category linking
// Returns: Boolean
function isContentNote(noteTag) {
	const contentTypes = ["Basic", "Tool", "TTP", "Playbook", "Mindmap", "Debrief", "Payload", "Infrastructure", "Study Resources"];
	return contentTypes.some(type => noteTag.startsWith(type));
}

//////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////// Main //////////////////////////////////////
//////////////////////////////////////////////////////////////////////////////////

// Present selection of Search Tags to user
const noteTag = await setTag();

// Ensure the user is not supplying an undefined or untitled note name
let title = tp.file.title;
title = await setTitle(title);

// Handle category selection based on note type
let selectedPrimaryCategories = [];
let selectedSecondaryCategories = [];

if (noteTag.startsWith("Secondary")) {
	selectedPrimaryCategories = await setCategories("primary");
} else if (isContentNote(noteTag)) {
	selectedPrimaryCategories = await setCategories("primary");
	selectedSecondaryCategories = await setCategories("secondary");
}

// Get the Struct associated with the Search Tag selected
const config = await getNoteStruct(noteTag, selectedPrimaryCategories, selectedSecondaryCategories);

// move file to appropriate corresponding folder
await tp.file.move(config.destination + "/" + config.prefix + title);

//build the note
const note = await buildNote(title,config);
%><%* tR += `${note}` %>
