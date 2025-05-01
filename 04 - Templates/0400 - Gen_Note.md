<%* 
//////////////////////////////////////////////////////////////////////////////////////////////////////////////////// Functions /////////////////////////////////////
//////////////////////////////////////////////////////////////////////////////////

// Name: setTitle
// Description: Ensure a valid title is supplied, or prompt user for title.
// Rename the created file from "Untitled" to what they've supplied.
// Supports both creating notes from "new note" or "ctrl + click"
async function setTitle() {

	if (await isValidTitle(title)) 
	{
		console.log(`Title: "${title}" is valid`)
		await tp.file.rename(title);
	}
	else 
	{
		console.log("The supplied title was invalid.")
		title = await tp.system.prompt("Title of New Note");
		setTitle();
	}
}

// Name: isValidTitle
// Description: Checks supplied string (title) against 'undefined' type , null 
// value, empty value and when the type is a string - we validate if it follows 
// the default naming scheme of 'Untitled #' - rejects assignment of any of these 
// cases
// Return: Boolean
async function isValidTitle(noteTitle) {
	// if nothing was supplied, try again
	if (typeof noteTitle === 'undefined' || (typeof noteTitle === 'string' && noteTitle.includes('Untitled')) || noteTitle === null || noteTitle === "") 
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
	var noteStructConfig;
	console.log("Getting Struct MD for tag: " + noteTag);
	
	if (tag.startsWith("Primary")) {
		console.log("Found Primary config");
		noteStructConfig = {
			name: "Primary",
			prefix: "01 - ",
			destination: "01 - Primary Categories/",
			metadata: "[[04 - Templates/04 - Primary Category/0401 - Metadata]]",
			body: "[[04 - Templates/04 - Primary Category/0402 - Body]]"

		};
	} else if (tag.startsWith("Secondary")) {
		console.log("Found Secondary config");
		noteStructConfig = {
			name: "Secondary",
			prefix: "02 - ",
			destination: "02 - Secondary Categories/",
			metadata: "[[04 - Templates/04 - Secondary Category/0401 - Metadata]]",
			body: "[[04 - Templates/04 - Secondary Category/0402 - Body]]"
		};
	} else if (tag.startsWith("Basic")) {
		console.log("Found Basic config");
		noteStructConfig = {
			name: "Basic",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040101 - Metadata]]",
			header: "[[040102 - Header]]",
			body: "[[040103 - Body]]"
		};
	} else if (tag.startsWith("Tool")) {
		console.log("Found Tool config");
		noteStructConfig = {
			name: "Tool",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040201 - Metadata]]",
			header: "[[040202 - Header]]",
			body: "[[040203 - Body]]"
		};
	} else if (tag.startsWith("TTP")) {
		console.log("Found TTP config");
		noteStructConfig = {
			name: "Basic",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040301 - Metadata]]",
			header: "[[040302 - Header]]",
			body: "[[040303 - Body]]"
		};
	} else if (tag.startsWith("Playbook")) {
		console.log("Found Playbook config");
		noteStructConfig = {
			name: "Playbook",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040401 - Metadata]]",
			header: "[[040402 - Header]]",
			body: "[[040403 - Body]]"
		};
	} else if (tag.startsWith("Mindmap")) {
		console.log("Found Mindmap config");
		noteStructConfig = {
			name: "Mindmap",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040501 - Metadata]]",
			header: "[[040502 - Header]]",
			body: "[[040503 - Body]]"
		};
	} else if (tag.startsWith("Debrief")) {
		console.log("Found Debrief config");
		noteStructConfig = {
			name: "Debrief",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040601 - Metadata]]",
			header: "[[040602 - Header]]",
			body: "[[040603 - Body]]"
		};
	} else if (tag.startsWith("Payload")) {
		console.log("Found Payload config");
		noteStructConfig = {
			name: "Payload",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040701 - Metadata]]",
			header: "[[040702 - Header]]",
			body: "[[040703 - Body]]"
		};
	} else if (tag.startsWith("Infrastructure")) {
		console.log("Found Infrastructure config");
		noteStructConfig = {
			name: "Infrastructure",
			prefix: "",
			destination: "03 - Content/",
			metadata: "[[040801 - Metadata]]",
			header: "[[040802 - Header]]",
			body: "[[040803 - Body]]"
		};
	} else {
		console.log("You selected an option outside what was expected.");
		console.log("Try again.");
	}
	return noteStructConfig;
	//return "test";
}

// Name: setTimestamps
// Description: Builds the timestamps element appended to note pages as they're built. Includes the definition of the Modified Date dynamic element which is updated when the page is edited and shown at the time of the page being rendered in preview mode.
async function setTimestamps() {
	var timestamps = "\nCreated Date: " + tp.file.creation_date('MMMM Do YYYY (HH:mm a)') + "  \nLast Modified Date: \<%+tp.file.last_modified_date(\"MMMM Do YYYY (HH:mm a)\")%\>";
	
	return timestamps;
}

// Name: isStructCategory
// Description: Determines whether or not the struct is related to a category
// by its name equaling "Primary" or "Secondary"
// Returns: Boolean
async function isStructCategory(structName) {
	// dont build out template elements, just reference
	if (structName == "Primary" || structName == "Secondary") {
		console.log("yes");
		return true;
	} 
	console.log("no");
	return false;
}

// Name: buildNote
// Description: Performs the actual concatenation of our discrete header, body and 
// footer note elements then handles the creation of the note itself.
async function buildNote(title,noteStructConfig) {
	
	// Notes:
	// - Leaving the search tag blank for now
	// - Headers are intentionally left mostly blank for now
	// - Timestamp has to be manually added as literal string; attempting to add timestamp
	// as a property or included file results in "NaN" rendering or other errors
	// - Removed setMetdata function; setting properties in included files is easier IMO
	// - Removed shouldLink function; purpose unclear, seems unnecessary
	var meta = await tp.file.include(noteStructConfig["metadata"]);
	var body = await tp.file.include(noteStructConfig["body"]);
	var time = await setTimestamps();
	var isCategory = await isStructCategory(noteStructConfig["name"]);
	
	if (isCategory === true) {
		note = meta + body + time;
	}
	else if (isCategory === false) {
		var head = await tp.file.include(noteStructConfig["header"]);
		note = meta + head + body + time;
	}
}

//////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////// Main //////////////////////////////////////
//////////////////////////////////////////////////////////////////////////////////

var note;
var config;
let title = tp.file.title;

// Present selection of Search Tags to user
let tag = await tp.system.suggester(["🥇 (Primary)", "🥈 (Secondary)", "📝 (Basic)", "⛏️ (Tool)", "📕 (TTP)", "✅ (Playbook)", "🗺️ (Mindmap)", "⌛ (Debrief)", "💣 (Payload)", "🏗️ (Infrastructure)"], ["Primary", "Secondary", "Basic", "Tool", "TTP", "Playbook", "Mindmap", "Debrief", "Payload", "Infrastructure"], true);

// Ensure the user is not supplying an undefined or untitled note name
await setTitle(title);

// Get the Struct associated with the Search Tag selected
config = await getNoteStruct(tag);

// move file to appropriate corresponding folder
await tp.file.move(config["destination"] + "/" + config["prefix"] + title);

//build the note
await buildNote(title,config);
%><%* tR += `${note}` %>
