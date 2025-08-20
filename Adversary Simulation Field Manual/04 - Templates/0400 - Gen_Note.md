<%* 
//////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////// Functions ////////////////////////////////////
//////////////////////////////////////////////////////////////////////////////////

// Name: setNoteType
// Description: Ensure a valid Note Type is selected, keep prompting until one is chosen
async function setNoteType() {
    let selectedNoteType = null;
    while (!selectedNoteType) {
        selectedNoteType = await tp.system.suggester(
            ["🥇 (Primary Category)", "🥈 (Secondary Category)", "⚛️ (Content/Atomic Note)"], 
            ["Primary", "Secondary", "Content"], 
            false,
            "Select note type (required):"
        );
        // If null/undefined, loop continues
        if (!selectedNoteType) {
	        new Notice("Note type selection is required to continue");
	    }
    }
    return selectedNoteType;
}

// Name: setTitle
// Description: Ensure a valid title is supplied, or prompt user for title.
// Rename the created file from "Untitled" to what they've supplied.
async function setTitle(title, noteType) {
    let destinationPath;
    let promptText;
    
    switch(noteType) {
        case "Primary":
            destinationPath = "01 - Primary Categories/01 - " + title + ".md";
            promptText = "Title of New Primary Category";
            break;
        case "Secondary":
            destinationPath = "02 - Secondary Categories/02 - " + title + ".md";
            promptText = "Title of New Secondary Category";
            break;
        case "Content":
            destinationPath = "03 - Content/" + title + ".md";
            promptText = "Title of New Content/Atomic Note";
            break;
    }
    
    if (await isValidTitle(title, destinationPath)) {
        console.log(`Title: "${title}" is valid`);
        return title;
    } else {
        const newTitle = await tp.system.prompt(promptText);
        return await setTitle(newTitle, noteType);
    }
}

// Name: isValidTitle
// Description: Checks supplied string (title) against 'undefined' type, null 
// value, empty value and when the type is a string - we validate if it follows 
// the default naming scheme of 'Untitled #' - rejects assignment of any of these 
// cases. Also checks for duplicates in destination directory.
// Return: Boolean
async function isValidTitle(title, destinationPath) {
    const noteExists = await tp.file.exists(destinationPath);
    
    // If nothing was supplied, try again
    if (typeof title === 'undefined' || (typeof title === 'string' && title.includes('Untitled')) || title === null || title === "") {
        new Notice("Provide a distinct note title to continue");
        return false;
    } 
    // If duplicate exists in destination directory, try again
    else if (noteExists === true) {
	    new Notice("A note with this title already exists in the destination directory");
        return false;
    }
    
    return true;
}

// Name: getAllUsedEmojis
// Description: Scans all Notes in the vault to find emojis already in use as Search Tags
// Returns: Set of used emojis
// TODO: Figure out why this is not working; most likely the emojiMatch regex
async function getAllUsedEmojis() {
    const usedEmojis = new Set();
    
    // Get all markdown files in the vault
    const allFiles = app.vault.getMarkdownFiles();
    
    for (const file of allFiles) {
        try {
            const content = await app.vault.read(file);
            const frontmatter = app.metadataCache.getFileCache(file)?.frontmatter;
            
            if (frontmatter && frontmatter.tags) {
                // Extract emojis from tags
                frontmatter.tags.forEach(tag => {
                    // Match emojis at the start of tags (excluding system tags like 🥇Primary_Category)
                    const emojiMatch = tag.match(/^([\u{1F600}-\u{1F64F}]|[\u{1F300}-\u{1F5FF}]|[\u{1F680}-\u{1F6FF}]|[\u{1F1E0}-\u{1F1FF}]|[\u{2600}-\u{26FF}]|[\u{2700}-\u{27BF}])/u);
                    if (emojiMatch && !tag.includes('Primary_Category') && !tag.includes('Secondary_Category')) {
                        usedEmojis.add(emojiMatch[1]);
                    }
                });
            }
        } catch (error) {
            console.log(`Error reading file ${file.path}: ${error}`);
        }
    }
    
    // Also check existing system emojis that are reserved
    const reservedEmojis = ['🥇', '🥈', '⚛️'];
    reservedEmojis.forEach(emoji => usedEmojis.add(emoji));
    
    return usedEmojis;
}

// Name: getCategorizedEmojis  
// Description: Returns emojis organized by red team categories
function getCategorizedEmojis() {
    return {
        "🔥 Attack & Exploitation": [
            {emoji: "💥", desc: "Attack/Boom"}, 
            {emoji: "💣", desc: "Payload/Exploit"},
            {emoji: "⚔️", desc: "Weapon/Tool"},
            {emoji: "🎯", desc: "Target/Precision"},
            {emoji: "🔨", desc: "Brute Force"},
            {emoji: "💉", desc: "Injection"},
            {emoji: "🎣", desc: "Phishing"},
            {emoji: "🎭", desc: "Impersonation/Social Engineering"},
            {emoji: "🐚", desc: "Shell Access"},
            {emoji: "🔴", desc: "Red Team"},
            {emoji: "🟣", desc: "Purple Team"}
        ],
        "🛡️ Defense & Security": [
            {emoji: "🛡️", desc: "Defense/Shield"},
            {emoji: "🔒", desc: "Secured/Locked"},
            {emoji: "🔐", desc: "Encryption"},
            {emoji: "🔑", desc: "Authentication"},
            {emoji: "🧱", desc: "Firewall/Block"},
            {emoji: "👁️", desc: "Monitoring"},
            {emoji: "⚠️", desc: "Warning"},
            {emoji: "🔵", desc: "Blue Team"},
            {emoji: "🚨", desc: "Alert"},
            {emoji: "🗝️", desc: "Old Key/Cryptography"}
        ],
        "📊 Analysis & Intelligence": [
            {emoji: "📊", desc: "Analysis/Charts"},
            {emoji: "🔍", desc: "Investigation"},
            {emoji: "📈", desc: "Metrics/Growth"},
            {emoji: "📋", desc: "Checklist/Audit"},
            {emoji: "📝", desc: "Documentation"},
            {emoji: "📚", desc: "Research/Learn"},
            {emoji: "🧠", desc: "Brain/Intelligence"},
            {emoji: "😈", desc: "Threat Actor"}
        ],
        "🌐 Network & Infrastructure": [
            {emoji: "🌐", desc: "Network/Web"},
            {emoji: "📡", desc: "Communication"},
            {emoji: "🔗", desc: "Links/Connections"},
            {emoji: "💻", desc: "Computer/Client"},
            {emoji: "🖥️", desc: "Server/Desktop"},
            {emoji: "📱", desc: "Mobile Device"},
            {emoji: "🗄️", desc: "Database/Storage"},
            {emoji: "📶", desc: "Wireless/Signal"}
        ],
        "🔧 Tools & Utilities": [
            {emoji: "🔧", desc: "Tool/Utility"},
            {emoji: "⚙️", desc: "Configuration"},
            {emoji: "🛠️", desc: "Toolkit/Setup"},
            {emoji: "🤖", desc: "Automation"},
            {emoji: "💾", desc: "Storage/Backup"},
            {emoji: "📦", desc: "Package/Bundle"},
            {emoji: "⚡", desc: "Fast/Performance"},
            {emoji: "💲", desc: "Dollar Sign/Command"}
        ],
        "➕ Miscellaneous": [
            {emoji: "💡", desc: "Light Bulb/Idea"},
            {emoji: "🏦", desc: "Bank/Vault"},
            {emoji: "👤", desc: "Silhouette/Person"},
            {emoji: "🕳️", desc: "Hole/Vulnerability"},
            {emoji: "👣", desc: "Footprint/Clue"},
            {emoji: "🧪", desc: "Test Tube/Lab"},
            {emoji: "💀", desc: "Skull"},
            {emoji: "📡", desc: "Satellite"},
            {emoji: "⌨️", desc: "Keyboard"},
            {emoji: "🖱️", desc: "Mouse"},
            {emoji: "💿", desc: "CD"},
            {emoji: "🔌", desc: "Electric Plug"},
            {emoji: "🔬", desc: "Microscope"},
            {emoji: "🔭", desc: "Telescope"},
            {emoji: "⚖️", desc: "Balance Scale"},
            {emoji: "🔗", desc: "Link"},
            {emoji: "🌍", desc: "Earth Globe Europe-Africa"}
        ]
    };
}

// Name: smartEmojiSelector
// Description: Main function for intelligent emoji selection
async function smartEmojiSelector(name) {
    const usedEmojis = await getAllUsedEmojis();
    
    // Build selection options - only categorized emojis
    const options = [];
    const displayOptions = [];
    
    // Add categorized options
    const categories = getCategorizedEmojis();
    for (const [categoryName, emojis] of Object.entries(categories)) {
        const availableInCategory = emojis.filter(e => !usedEmojis.has(e.emoji));
        
        availableInCategory.forEach(item => {
            const display = `${item.emoji} ${item.desc}`;
            displayOptions.push(display);
            options.push(item.emoji);
        });
    }
    
    // Add manual entry and random options
    displayOptions.push("✏️ Enter emoji manually");
    options.push("MANUAL_ENTRY");
    displayOptions.push("🎲 Random available emoji");
    options.push("RANDOM");
    
    // Verify arrays match
    console.log(`Display options count: ${displayOptions.length}, Options count: ${options.length}`);
    
    // Show selection
    const promptText = `Select emoji for "${name}":`;
    
    const selection = await tp.system.suggester(
        displayOptions,
        options,
        false,
        promptText
    );
    
    if (!selection) {
	    new Notice(`Search tag selection failed; using 📁 as fallback search tag`);
        return "📁";
    }
    
    // Handle special selections
    if (selection === "MANUAL_ENTRY") {
        return await handleManualEmojiEntry();
    }
    
    if (selection === "RANDOM") {
        const allEmojis = Object.values(categories).flat().map(e => e.emoji);
        const available = allEmojis.filter(e => !usedEmojis.has(e));
        return available[Math.floor(Math.random() * available.length)] || "📁";
    }
    
    return selection;
}

// Name: handleManualEmojiEntry
// Description: Handles manual emoji entry with validation
// Returns: Validated emoji string
async function handleManualEmojiEntry() {
	let manualEmoji = null;
	
	// Basic validation - check if it looks like an emoji
    const emojiRegex = /[\u{1F600}-\u{1F64F}]|[\u{1F300}-\u{1F5FF}]|[\u{1F680}-\u{1F6FF}]|[\u{1F1E0}-\u{1F1FF}]|[\u{2600}-\u{26FF}]|[\u{2700}-\u{27BF}]/u;
    
    while (!manualEmoji) {
        manualEmoji = await tp.system.prompt("Enter emoji character (paste from system emoji picker):");
        // If null/undefined/invalid, loop continues
        if (!manualEmoji || !emojiRegex.test(manualEmoji)) {
	        const retry = await tp.system.prompt("That doesn't look like an emoji. Try again? (y/n)");
	        if (retry?.toLowerCase() === 'y') {
	            continue;
	        }
	        new Notice(`Manual entry failed; using 📁 as fallback search tag`);
	        return "📁";
	    }
    }
    
    // Check if already in use
    // TODO: Emoji search is not working, troubleshoot before attempting to enforce emoji uniqueness
    /*
    const usedEmojis = await getAllUsedEmojis();
    if (usedEmojis.has(manualEmoji)) {
        const retry = await tp.system.prompt(`Emoji ${manualEmoji} is already in use. Try another? (y/n)`);
        if (retry?.toLowerCase() === 'y') {
            return await handleManualEmojiEntry();
        }
        throw new Error("Emoji already in use");
    }
    */
    
    return manualEmoji;
}

// Name: getPrimaryCategories
// Description: Gets list of all primary category files in the vault
// Returns: Array of primary category names (without prefixes)
async function getPrimaryCategories() {
    const primaryCatFolder = app.vault.getAbstractFileByPath("01 - Primary Categories");
    if (!primaryCatFolder) {
        return [];
    }
    
    const files = primaryCatFolder.children
        .filter(file => file.extension === "md")
        .map(file => file.basename.replace(/^01 - /, "")); // Remove "01 - " prefix
    
    return files.sort(); // Return in alphabetical order
}

// Name: getSecondaryCategories
// Description: Gets list of all secondary category files in the vault
// Returns: Array of secondary category names (without prefixes)
async function getSecondaryCategories() {
    const secondaryCatFolder = app.vault.getAbstractFileByPath("02 - Secondary Categories");
    if (!secondaryCatFolder) {
        return [];
    }
    
    const files = secondaryCatFolder.children
        .filter(file => file.extension === "md")
        .map(file => file.basename.replace(/^02 - /, "")); // Remove "02 - " prefix
    
    return files.sort(); // Return in alphabetical order
}

// Name: selectCategories
// Description: Prompts user to select one or more categories (Primary or Secondary)
// Returns: Array of selected category names with proper linking format
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

// Name: getAvailableContentTypes
// Description: Gets list of all available Content Types from templates with their Search Tag emojis
// Returns: Array of Content Type objects with name, template info, and emoji from metadata
async function getAvailableContentTypes() {
    const contentTypesFolder = app.vault.getAbstractFileByPath("04 - Templates/04 - Content");
    if (!contentTypesFolder) {
        return [];
    }
    
    const contentTypes = [];
    
    for (const folder of contentTypesFolder.children) {
        if (folder.children) { // Only folders
            const match = folder.name.match(/^(\d{4}) - (.+)$/);
            if (match) {
                const templateNumber = match[1];
                const typeName = match[2];
                
                // Try to extract emoji from metadata file
                let emoji = "📝"; // Default emoji if none found
                try {
                    const metadataPath = `04 - Templates/04 - Content/${folder.name}/${templateNumber}01 - Metadata.md`;
                    const metadataFile = app.vault.getAbstractFileByPath(metadataPath);
                    
                    if (metadataFile) {
                        const content = await app.vault.read(metadataFile);
                        const frontmatter = app.metadataCache.getFileCache(metadataFile)?.frontmatter;
                        
                        if (frontmatter && frontmatter.tags && Array.isArray(frontmatter.tags)) {
                            // Find the content type tag (not system tags like Primary_Category)
                            const contentTag = frontmatter.tags.find(tag => 
                                !tag.includes('Primary_Category') && 
                                !tag.includes('Secondary_Category') &&
                                typeof tag === 'string'
                            );
                            
                            if (contentTag) {
                                // Extract emoji from the beginning of the tag
                                const emojiMatch = contentTag.match(/^([\u{1F600}-\u{1F64F}]|[\u{1F300}-\u{1F5FF}]|[\u{1F680}-\u{1F6FF}]|[\u{1F1E0}-\u{1F1FF}]|[\u{2600}-\u{26FF}]|[\u{2700}-\u{27BF}])/u);
                                if (emojiMatch) {
                                    emoji = emojiMatch[1];
                                }
                            }
                        }
                    }
                } catch (error) {
                    console.log(`Error reading metadata for ${typeName}: ${error}`);
                    // Keep default emoji
                }
                
                contentTypes.push({
                    name: typeName,
                    emoji: emoji,
                    templateNumber: templateNumber,
                    displayName: `${emoji} (${typeName})`,
                    searchTag: `${emoji}${typeName.replace(/\s+/g, '_')}`
                });
            }
        }
    }
    
    // Sort alphabetically by name
    return contentTypes.sort((a, b) => a.name.localeCompare(b.name));
}

// Name: setContentType
// Description: Prompts user to select Content Type or create new one, displaying emoji and name
// Returns: Content Type configuration object
async function setContentType() {
    const availableTypes = await getAvailableContentTypes();
    
    // Create options array
    const options = [];
    const displayOptions = [];
    
    // Add existing Content Types with emoji display
    availableTypes.forEach(type => {
        options.push(type);
        displayOptions.push(type.displayName); // Now includes emoji: "💣 (Payload)"
    });
    
    // Add "Create New Content Type" option
    options.push("NEW_CONTENT_TYPE");
    displayOptions.push("➕ Create New Content Type");

	let selectedContentType = null;
	while (!selectedContentType) {
        selectedContentType = await tp.system.suggester(
	        displayOptions,
	        options,
	        false,
	        "Select Content Type or create new one:"
	    );
        // If null/undefined, loop continues
        if (!selectedContentType) {
	        new Notice("Content type selection is required");
        }
    }
    
    if (selectedContentType === "NEW_CONTENT_TYPE") {
        return await createNewContentType();
    }
    
    return selectedContentType;
}

// Name: getNextTemplateNumber
// Description: Finds the next available template number in the 04 - Content folder
// Returns: String with next template number (e.g., "0410")
async function getNextTemplateNumber() {
    const templatesFolder = app.vault.getAbstractFileByPath("04 - Templates/04 - Content");
    if (!templatesFolder) {
        return "0410"; // Start at 0410 if no content templates exist
    }
    
    const existingNumbers = templatesFolder.children
        .filter(child => child.children) // Only folders
        .map(folder => {
            const match = folder.name.match(/^(\d{4})/);
            return match ? parseInt(match[1]) : 0;
        })
        .filter(num => num > 0);
    
    if (existingNumbers.length === 0) {
        return "0410";
    }
    
    const maxNumber = Math.max(...existingNumbers);
    return String(maxNumber + 1).padStart(4, '0');
}

// Name: createNewContentType
// Description: Creates a new Content Type template structure with unique emoji validation
// Returns: Object with new Content Type configuration
async function createNewContentType() {
    // Get the content type name
    let contentTypeName = null;
	while (!contentTypeName) {
        contentTypeName = await tp.system.prompt("Enter name for new content type (e.g., 'Report', 'Research'):");
        // If null/undefined, loop continues
        if (!contentTypeName) {
	        new Notice("Content type name is required");
        }
    }
    
    // Get unique emoji for the Content Type
    const selectedEmoji = await smartEmojiSelector(contentTypeName);
    
    // Generate next available template number
    const templateNumber = await getNextTemplateNumber();
    
    // Create the template directory structure by copying Basic template
    await createTemplateStructure(templateNumber, contentTypeName, selectedEmoji);
    
    return {
        name: contentTypeName,
        emoji: selectedEmoji,
        templateNumber: templateNumber,
        displayName: `${contentTypeName} (${templateNumber})`,
        tag: `${selectedEmoji}${contentTypeName}`
    };
}

// Name: createTemplateStructure
// Description: Creates the folder structure and template files for a new Content Type by copying Basic
async function createTemplateStructure(templateNumber, contentTypeName, emoji) {
    const basePath = `04 - Templates/04 - Content/${templateNumber} - ${contentTypeName}`;
    const basicPath = "04 - Templates/04 - Content/0401 - Basic";
    
    // Create the folder structure
    try {
        await app.vault.createFolder(basePath);
    } catch (error) {
        console.log(`Folder creation note: ${error.message}`);
        // Folder might already exist, continue
    }
    
    // Copy and modify templates from Basic
    const templateFiles = [
        { source: "040101 - Metadata.md", target: `${templateNumber}01 - Metadata.md` },
        { source: "040102 - Header.md", target: `${templateNumber}02 - Header.md` },
        { source: "040103 - Body.md", target: `${templateNumber}03 - Body.md` }
    ];
    
    console.log(`Creating template files in: ${basePath}`);
    console.log(`Copying from: ${basicPath}`);
    
    for (const fileInfo of templateFiles) {
        try {
            const sourceFilePath = `${basicPath}/${fileInfo.source}`;
            const targetFilePath = `${basePath}/${fileInfo.target}`;
            
            console.log(`Processing: ${fileInfo.source} -> ${fileInfo.target}`);
            
            const sourceFile = app.vault.getAbstractFileByPath(sourceFilePath);
            if (!sourceFile) {
                console.log(`Source file not found: ${sourceFilePath}`);
                continue;
            }
            
            let content = await app.vault.read(sourceFile);
            console.log(`Read content from ${fileInfo.source}, length: ${content.length}`);
            
            // Modify metadata file to include new emoji and type
            if (fileInfo.source.includes("Metadata")) {
                content = content.replace(
                    'tags:\n  - 📝Basic',
                    `tags:\n  - ${emoji}${contentTypeName.replace(/\s+/g, '_')}` // Replace spaces with underscores
                );
                content = content.replace(
                    'type: Basic',
                    `type: ${contentTypeName}`
                );
                console.log(`Modified metadata content for ${contentTypeName}`);
            }
            
            // Check if target file already exists
            const existingFile = app.vault.getAbstractFileByPath(targetFilePath);
            if (existingFile) {
                console.log(`File already exists, overwriting: ${targetFilePath}`);
                await app.vault.modify(existingFile, content);
            } else {
                // Use app.vault.create instead of tp.file.create_new for more reliable creation
                console.log(`Creating new file: ${targetFilePath}`);
                await app.vault.create(targetFilePath, content);
            }
            
            console.log(`Successfully created: ${fileInfo.target}`);
            
        } catch (error) {
            console.log(`Error creating template file ${fileInfo.target}: ${error}`);
            console.log(`Error details:`, error);
            new Notice(`Failed to create ${fileInfo.target}: ${error.message}`);
        }
    }
    
    new Notice(`Created new content type: ${emoji} ${contentTypeName}`);
    new Notice("Template files created and ready to customize!");
}

// Name: getNoteConfig
// Description: Builds configuration object for note creation based on note type
// Returns: Configuration object for note building
async function getNoteConfig(noteType, title, primaryCategories = [], secondaryCategories = [], contentTypeConfig = null, emoji = null) {
    let config = {
        noteType: noteType,
        title: title,
        primaryCategories: primaryCategories,
        secondaryCategories: secondaryCategories
    };
    
    switch(noteType) {
        case "Primary":
	        console.log("Primary");
            config.prefix = "01 - ";
            config.destination = "01 - Primary Categories/";
            config.metadataTemplate = "[[04 - Templates/04 - Primary Category/0401 - Metadata]]";
            config.bodyTemplate = "[[04 - Templates/04 - Primary Category/0402 - Body]]";
            config.emoji = emoji;
            break;
            
        case "Secondary":
            config.prefix = "02 - ";
            config.destination = "02 - Secondary Categories/";
            config.metadataTemplate = "[[04 - Templates/04 - Secondary Category/0401 - Metadata]]";
            config.bodyTemplate = "[[04 - Templates/04 - Secondary Category/0402 - Body]]";
            break;
            
        case "Content":
            config.prefix = "";
            config.destination = "03 - Content/";
            config.contentType = contentTypeConfig;
            config.metadataTemplate = `[[04 - Templates/04 - Content/${contentTypeConfig.templateNumber} - ${contentTypeConfig.name}/${contentTypeConfig.templateNumber}01 - Metadata]]`;
            config.headerTemplate = `[[04 - Templates/04 - Content/${contentTypeConfig.templateNumber} - ${contentTypeConfig.name}/${contentTypeConfig.templateNumber}02 - Header]]`;
            config.bodyTemplate = `[[04 - Templates/04 - Content/${contentTypeConfig.templateNumber} - ${contentTypeConfig.name}/${contentTypeConfig.templateNumber}03 - Body]]`;
            break;
    }
    
    return config;
}

// Name: buildNote
// Description: Creates the actual note content based on configuration
// Returns: String containing the complete note content
async function buildNote(config) {
    // Load metadata template
    let metadata = await tp.file.include(config.metadataTemplate);

    // Handle metadata modifications based on note type
    switch(config.noteType) {
        case "Primary":
            if (config.emoji) {
                metadata = metadata.replace(
                    'tags:\n  - 🥇Primary_Category\n  - ADD_NEW_PRIMARY_CATEGORY_EMOJI',
                    `tags:\n  - 🥇Primary_Category\n  - ${config.emoji}${config.title.replace(/\s+/g, '_')}` // Replace spaces with underscores
                );
            }
            break;
            
        case "Secondary":
            if (config.primaryCategories.length > 0) {
                const primaryCatList = config.primaryCategories.join('\n  - ');
                metadata = metadata.replace(
                    'primary categories:\n  - Add link(s) [[]] back to related PRIMARY categories',
                    `primary categories:\n  - ${primaryCatList}`
                );
            }
            break;
            
        case "Content":
            if (config.primaryCategories.length > 0) {
                const primaryCatList = config.primaryCategories.join('\n  - ');
                metadata = metadata.replace(
                    'primary categories:\n  - Add link(s) [[]] back to related PRIMARY categories',
                    `primary categories:\n  - ${primaryCatList}`
                );
            }
            
            if (config.secondaryCategories.length > 0) {
                const secondaryCatList = config.secondaryCategories.join('\n  - ');
                metadata = metadata.replace(
                    'secondary categories:\n  - Add link(s) [[]] back to related SECONDARY categories',
                    `secondary categories:\n  - ${secondaryCatList}`
                );
            }
            break;
    }
    
    // Build page title
    const pageTitle = `# [[${config.prefix}${config.title}]]\n`;
    
    // Load body template
    const body = await tp.file.include(config.bodyTemplate);
    
    // Add timestamps
    const timestamps = "\n*Created Date*: <%+tp.file.creation_date(\"MMMM Do YYYY (HH:mm a)\")%\>  \n*Last Modified Date*: \<%+tp.file.last_modified_date(\"MMMM Do YYYY (HH:mm a)\")%\>";
    
    // For content notes, include header
    if (config.noteType === "Content") {
        const header = await tp.file.include(config.headerTemplate);
        return metadata + pageTitle + header + body + timestamps;
    } else {
        return metadata + pageTitle + body + timestamps;
    }
}

//////////////////////////////////////////////////////////////////////////////////
////////////////////////////////////// Main //////////////////////////////////////
//////////////////////////////////////////////////////////////////////////////////

// Definitions:
// Note - A file created in either the Primary Category, Secondary Category, or Content sub-directories
// Note Type - A Primary Category, Secondary Category, or Content Note
// Primary Category - The highest-level form of a note in Zettelkasten (e.g., "Penetration Test", "Red Team", or "Vault Administration"); Primary Categories are stored in the "01 - Primary Categories" directory, built with the "04 - Templates/04 - Primary Categories" template structures, have the "🥇Primary_Category" Search Tag, and individual Primary Categories have their own additional Search Tag (e.g., the "Penetration Test" Primary Category has the Search Tags "🥇Primary_Category" and "🎯Penetration_Test" Search Tags)
// Secondary Category - An intermediate-level form of a note in Zettelkasten (e.g., "Active Directory", "Metasploit", or "Post-Exploitation"); Secondary Categories are stored in the "02 - Secondary Categories" directory, built with the "04 - Templates/04 - Secondary Categories" template structures, and only have the "🥈Secondary_Category" Search Tag
// Content/Atomic Note - The lowest-level form of a note in Zettelkasten (e.g., "Admonition", "Obsidian - Linking Content", or "Templater"); Content Notes are stored in the "03 - Content" directory, built with their own individual template structures (e.g., a "Basic" Content Type is built with the structures located at "04 - Templates/04 - Content/0401 - Basic"), and are assigned a Search Tag based on the Content Type used to originally build the note (e.g., the "Basic" Content Type has the Search Tag "📝Basic")
// Content Type - A general framework for building Content/Atomic Notes that leverages individualized template structures in the "04 - Templates/04 - Content" directory
// Search Tag - A unique emoji postfixed with the Note title, replacing whitespace characters with underscore characters (e.g., "🥇Primary_Category", "✍️Reporting", or "💣Payload")

let noteContent = "";

try {
	// 1. User selects Note Type. Options are:
	// - 🥇 (Primary Category) -> "Primary"
	// - 🥈 (Secondary Category) -> "Secondary"
	// - ⚛️ (Content/Atomic Note) -> "Content"
	const noteType = await setNoteType();
	
	switch(noteType) {
		case "Primary":
			// 1.1. Primary Category workflow
            let primaryTitle = tp.file.title;

            // 1.1.1. Get valid title
            primaryTitle = await setTitle(primaryTitle, noteType);
	
            // 1.1.2. Get emoji for category
            const categoryEmoji = await smartEmojiSelector(primaryTitle);
            
            // 1.1.3. Build config and create Note
            config = await getNoteConfig(noteType, primaryTitle, [], [], null, categoryEmoji);
            break;
    
		case "Secondary":
            // 1.2. Secondary Category workflow
            let secondaryTitle = tp.file.title;
		
            // 1.2.1. Get valid title
            secondaryTitle = await setTitle(secondaryTitle, noteType);
            
            // 1.2.2. Select Primary Categories to link back to
            const linkedPrimaryCategories = await selectCategories("primary");
            
            // 1.2.3. Build config and create Note
            config = await getNoteConfig(noteType, secondaryTitle, linkedPrimaryCategories, []);
            break;
            
		case "Content":
            // 1.3. Content/Atomic Note workflow
            let contentTitle = tp.file.title;
            
            // 1.3.1. Get valid title
            contentTitle = await setTitle(contentTitle, noteType);
            
            // 1.3.2. Select Content Type or create new one
            const selectedContentType = await setContentType();
            
            // 1.3.3. Select Primary Categories to link back to
            const contentPrimaryCategories = await selectCategories("primary");
            
            // 1.3.4. Select Secondary Categories to link back to
            const contentSecondaryCategories = await selectCategories("secondary");
            
            // 1.3.5. Build Note config
            config = await getNoteConfig(noteType, contentTitle, contentPrimaryCategories, contentSecondaryCategories, selectedContentType);
            break;
	}

	// 2. Move file to appropriate destination
    await tp.file.move(config.destination + config.prefix + config.title);

	// 3. Build and return the Note content
    noteContent = await buildNote(config);
    
} catch (error) {
    new Notice(`Error creating note: ${error.message}`);
    console.log("Note creation error:", error);
}

%><%* tR += `${noteContent}` %>
