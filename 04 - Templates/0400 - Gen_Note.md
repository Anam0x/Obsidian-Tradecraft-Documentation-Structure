<%* 
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////// Functions /////////////////////////////////////
//////////////////////////////////////////////////////////////////////////////////

// Name: setTitle
// Description: Ensure a valid title is supplied, or prompt user for title.
// Rename the created file from "Untitled" to what they've supplied.
// Supports both creating notes from "new note" or "ctrl + click"
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
	} else if (noteTag.startsWith("Secondary")) {
		noteStructConfig = {
			name: "Secondary",
			prefix: "02 - ",
			destination: "02 - Secondary Categories/",
			metadata: "[[04 - Templates/04 - Secondary Category/0401 - Metadata]]",
			body: "[[04 - Templates/04 - Secondary Category/0402 - Body]]"
		};
	} else if (noteTag.startsWith("Basic")) {
		noteStructConfig = {
			name: "Basic",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040101 - Metadata]]",
			header: "[[040102 - Header]]",
			body: "[[040103 - Body]]"
		};
	} else if (noteTag.startsWith("Tool")) {
		noteStructConfig = {
			name: "Tool",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040201 - Metadata]]",
			header: "[[040202 - Header]]",
			body: "[[040203 - Body]]"
		};
	} else if (noteTag.startsWith("TTP")) {
		noteStructConfig = {
			name: "Basic",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040301 - Metadata]]",
			header: "[[040302 - Header]]",
			body: "[[040303 - Body]]"
		};
	} else if (noteTag.startsWith("Playbook")) {
		noteStructConfig = {
			name: "Playbook",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040401 - Metadata]]",
			header: "[[040402 - Header]]",
			body: "[[040403 - Body]]"
		};
	} else if (noteTag.startsWith("Mindmap")) {
		noteStructConfig = {
			name: "Mindmap",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040501 - Metadata]]",
			header: "[[040502 - Header]]",
			body: "[[040503 - Body]]"
		};
	} else if (noteTag.startsWith("Debrief")) {
		noteStructConfig = {
			name: "Debrief",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040601 - Metadata]]",
			header: "[[040602 - Header]]",
			body: "[[040603 - Body]]"
		};
	} else if (noteTag.startsWith("Payload")) {
		noteStructConfig = {
			name: "Payload",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040701 - Metadata]]",
			header: "[[040702 - Header]]",
			body: "[[040703 - Body]]"
		};
	} else if (noteTag.startsWith("Infrastructure")) {
		noteStructConfig = {
			name: "Infrastructure",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040801 - Metadata]]",
			header: "[[040802 - Header]]",
			body: "[[040803 - Body]]"
		};
	} else if (noteTag.startsWith("Education")) {
		noteStructConfig = {
			name: "Education",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040901 - Metadata]]",
			header: "[[040902 - Header]]",
			body: "[[040903 - Body]]"
		};
	} else {
		console.log("You selected an option outside what was expected.");
		console.log("Try again.");
		const newNoteTag = await tp.system.suggester(["🥇 (Primary Category)", "🥈 (Secondary Category)", "📝 (Basic)", "⛏️ (Tool)", "📕 (TTP)", "✅ (Playbook)", "🗺️ (Mindmap)", "⌛ (Debrief)", "💣 (Payload)", "🏗️ (Infrastructure)", "🎓 (Education)"], ["Primary", "Secondary", "Basic", "Tool", "TTP", "Playbook", "Mindmap", "Debrief", "Payload", "Infrastructure", "Education"], true);
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

//////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////// Main //////////////////////////////////////
//////////////////////////////////////////////////////////////////////////////////

// Present selection of Search Tags to user
const noteTag = await tp.system.suggester(["🥇 (Primary Category)", "🥈 (Secondary Category)", "📝 (Basic)", "⛏️ (Tool)", "📕 (TTP)", "✅ (Playbook)", "🗺️ (Mindmap)", "⌛ (Debrief)", "💣 (Payload)", "🏗️ (Infrastructure)", "🎓 (Education)"], ["Primary", "Secondary", "Basic", "Tool", "TTP", "Playbook", "Mindmap", "Debrief", "Payload", "Infrastructure", "Education"], true);

// Ensure the user is not supplying an undefined or untitled note name
let title = tp.file.title;
title = await setTitle(title);

// Get the Struct associated with the Search Tag selected
const config = await getNoteStruct(noteTag);

// move file to appropriate corresponding folder
await tp.file.move(config.destination + "/" + config.prefix + title);

//build the note
const note = await buildNote(title,config);
%><%* tR += `${note}` %>
