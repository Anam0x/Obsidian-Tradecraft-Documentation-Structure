<%* 
/**
 * Adversary Simulation Field Manual - Automated Note and Category Creation System
 * 
 * This script automates the creation of hierarchical notes for a red team reference guide:
 * - Primary Categories (🥇): High-level topics like "Penetration Test", "Red Team"
 * - Secondary Categories (🥈): Mid-level topics like "Active Directory", "Metasploit" 
 * - Content Notes (⚛️): Atomic notes with specific content types like "Tools", "TTPs", "Payloads"
 * 
 * Features:
 * - Interactive note type selection with validation
 * - Smart emoji selection for categorization tags
 * - Template-based content generation
 * - Automated file organization and linking
 * - Dynamic content type and primary category creation
 */

//////////////////////////////////////////////////////////////////////////////////
//                                 CONSTANTS                                   //
//////////////////////////////////////////////////////////////////////////////////

const PATHS = {
    PRIMARY_CATEGORIES: "01 - Primary Categories",
    SECONDARY_CATEGORIES: "02 - Secondary Categories", 
    CONTENT: "03 - Content",
    CONTENT_TEMPLATES: "04 - Templates/04 - Content",
    PRIMARY_TEMPLATE_META: "[[04 - Templates/04 - Primary Category/Metadata]]",
    PRIMARY_TEMPLATE_BODY: "[[04 - Templates/04 - Primary Category/Body]]",
    SECONDARY_TEMPLATE_META: "[[04 - Templates/04 - Secondary Category/Metadata]]",
    SECONDARY_TEMPLATE_BODY: "[[04 - Templates/04 - Secondary Category/Body]]",
    BASIC_TEMPLATE: "04 - Templates/04 - Content/Basic"
};

const NOTE_TYPES = {
    PRIMARY: "Primary Category",
    SECONDARY: "Secondary Category", 
    CONTENT: "Content",
    CONTENT_TYPE: "Content Type"
};

const EMOJI_SELECTION = {
    TYPES: {
        UTILITIES: "UTILITIES_",
        MANUAL: "MANUAL_ENTRY", 
        RANDOM: "RANDOM"
    },
    DISPLAY_TEXT: {
	    UTILITIES: "--- Utilities ---",
        MANUAL: "✏️ Enter emoji manually",
        RANDOM: "🎲 Random selection"
    },
    DEFAULT_EMOJI: "📁"
};

const RESERVED_EMOJIS = new Set(['🥇', '🥈', '⚛️']);

const ILLEGAL_CHARS = /[<>:"/\\|?*\x00-\x1F]/g;

const INVISIBLE_CHARS = /[\u200B-\u200F\u202A-\u202E\u2060-\u206F\uFEFF]/;

const RESERVED_NAMES = /^(CON|PRN|AUX|NUL|COM[1-9]|LPT[1-9])(\.|$)/i;

const EMOJI_REGEX = /[\u{1F600}-\u{1F64F}]|[\u{1F300}-\u{1F5FF}]|[\u{1F680}-\u{1F6FF}]|[\u{1F1E0}-\u{1F1FF}]|[\u{2600}-\u{26FF}]|[\u{2700}-\u{27BF}]|[\u{1F900}-\u{1F9FF}]|[\u{1FA70}-\u{1FAFF}]|[\u{2300}-\u{23FF}]|[\u{2B50}]|[\u{2194}-\u{21AA}]|[\u{231A}-\u{231B}]|[\u{25AA}-\u{25FE}]/u;

const DIVIDER = "\n***\n";

const TIMESTAMP = "*Created Date*: <%+tp.file.creation_date(\"MMMM Do YYYY (HH:mm a)\")%\>  \n*Last Modified Date*: \<%+tp.file.last_modified_date(\"MMMM Do YYYY (HH:mm a)\")%\>";

//////////////////////////////////////////////////////////////////////////////////
//                              UTILITY FUNCTIONS                              //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Shows a user notice with consistent formatting
 * @param {string} message - Message to display
 */
function showNotice(message) {
    new Notice(message);
}

/**
 * Logs error with context and shows user-friendly notice
 * @param {Error} error - Error object
 * @param {string} context - Context description
 * @param {string} fallback - Fallback value description
 */
function logError(error, context, fallback = null) {
    console.error(`[Red Team Script] ${context}:`, error);
    const message = fallback 
        ? `${context} failed. Using ${fallback} as fallback.`
        : `Error in ${context}: ${error.message}`;
    showNotice(message);
}

/**
 * Enhanced validation for note titles with comprehensive error handling
 * @param {string} title - Title to validate
 * @param {string} destinationPath - Full path where note would be created
 * @param {boolean} isNote - Boolean indicating whether title is for a future note (true) or content type (false)
 * @returns {Promise<{isValid: boolean, error?: string, sanitized?: string}>} - Validation result with optional sanitized version
 */
async function isValidTitle(title, destinationPath, isNote) {
    const itemType = isNote ? "note" : "content type";
    const trimmedTitle = title.trim();
    
    // Check for empty, undefined, or default titles
    if (!title || 
        title.includes('Untitled') || 
        trimmedTitle === "") {
        return {
            isValid: false,
            error: `Title cannot be empty or be the default "Untitled" value`,
            suggestion: `Provide a distinct ${itemType} title`
        };
    }

	// Check for leading dots
	if (title.startsWith('.')) {
        return {
            isValid: false,
            error: `Title cannot start with a dot (.) character`,
            suggestion: `Try: "${trimmedTitle.substring(1) || 'Hidden File'}" or add a prefix`
        };
    }

	// Check for illegal filename characters
    const illegalMatches = trimmedTitle.match(ILLEGAL_CHARS);
    if (illegalMatches) {
        const uniqueChars = [...new Set(illegalMatches)];
        return {
            isValid: false,
            error: `Title contains characters that aren't allowed in filenames: ${uniqueChars.join(', ')}`,
            suggestion: `Replace these characters with spaces, hyphens, or underscores`
        };
    }

	// Check for Windows reserved names (case-insensitive)
    if (RESERVED_NAMES.test(trimmedTitle)) {
        return {
            isValid: false,
            error: `"${trimmedTitle}" is reserved by Windows and cannot be used as a filename`,
            suggestion: `Try adding a prefix or suffix, like "${trimmedTitle} Notes" or "My ${trimmedTitle}"`
        };
    }
	
	// Check for excessively long titles (to encourage conciseness, the character limit is currently 100, which is well below the typical 255-byte filename limit of most OSs)
	if (title.length > 100) {
        return {
            isValid: false,
            error: `Title is too long (${title.length} characters, maximum is 100)`,
            suggestion: `Try shortening it or using abbreviations`
        };
    }
    
	// Check for trailing dots and whitespace characters (problematic on Windows)
	if (title !== title.replace(/[. ]+$/, '')) {
        return {
            isValid: false,
            error: `Title cannot end with dots or spaces (Windows compatibility issue)`,
            suggestion: `Remove the trailing characters or replace with underscores`
        };
    }

	// Check for emojis mixed with text (might be intentional but worth flagging; can cause file explorer issues)
	const hasEmojis = EMOJI_REGEX.test(trimmedTitle);
	
	if (hasEmojis) {
	    return {
	        isValid: false,
	        error: `Title contains both emojis and text: "${trimmedTitle}"`,
	        suggestion: `Use text-only titles for consistency and cross-platform compatability`,
	        canProceedAnyway: false
	    };
	}

	// Check for invisible characters
	if (INVISIBLE_CHARS.test(trimmedTitle)) {
	    return {
	        isValid: false,
	        error: `Title contains invisible characters that may cause file system issues`,
	        suggestion: `Please retype the title to remove any hidden characters`
	    };
	}

    // Check for duplicates
    try {
        const noteExists = await tp.file.exists(destinationPath);
        if (noteExists) {
	        console.log("detected dupliacte note");
            return {
                isValid: false,
                error: `A ${itemType} with this title already exists in the destination directory`,
                suggestion: `Try adding a number, date, or descriptive suffix`
            };
        }
    } catch (error) {
        console.warn(`Could not check for duplicate file: ${error.message}`);
        // Continue with validation rather than failing
    }

	console.log("no issues found");
    return { isValid: true };
}

/**
 * Validates emoji for use in Obsidian tags with comprehensive error handling
 * @param {string} emoji - Emoji string to validate
 * @returns {Object} - Validation result with error messages and suggestions
 */
function isValidEmoji(emoji) {
    // Check for empty or invalid input
    if (!emoji || emoji.length === 0) {
        return {
            isValid: false,
            error: "Emoji cannot be empty",
            suggestion: "Enter a single emoji character"
        };
    }
    
    // Check if input matches emoji regular expression
    if (!EMOJI_REGEX.test(emoji)) {
        return {
            isValid: false,
            error: "Not a valid emoji",
            suggestion: "Try copying an emoji from your system's emoji picker"
        };
    }

    // Check for Zero Width Joiner sequences (problematic for Obsidian tags)
    const codepoints = Array.from(emoji).map(char => char.codePointAt(0));
    const hasZWJ = codepoints.includes(0x200D);
    
    if (hasZWJ) {
        return {
            isValid: false,
            error: "Emoji contains Zero Width Joiner sequences that may cause tag display issues in Obsidian",
            suggestion: "Try a simpler emoji without profession/family modifiers (e.g., 👨‍💻 → 💻, 🕵️‍♂️ → 🕵️)",
            canProceedAnyway: true // Flag to indicate user can override this warning
        };
    }

	// Check for invisible characters
	if (INVISIBLE_CHARS.test(emoji)) {
	    return {
	        isValid: false,
	        error: `Input contains invisible characters that may cause tag issues in Obsidian`,
	        suggestion: `Enter a single emoji character`
	    };
	}

	// Check if input contains whitespace characters
    if (emoji !== emoji.trim()) { 
	    return {
		    isValid: false,
		    error: "Input cannot contain whitespace characters",
		    suggestion: "Enter a single emoji character"
	    }
    }

	// Check if input contains a single visual character
	const segmenter = new Intl.Segmenter('en', { granularity: 'grapheme' });
	const segments = Array.from(segmenter.segment(emoji));
	
	if (segments.length > 1) {
	    return {
	        isValid: false,
	        error: "Input cannot contain multiple characters", 
	        suggestion: "Enter a single emoji character"
	    };
	}

	// Check if emoji is reserved for vault administration
	if (RESERVED_EMOJIS.has(emoji)) {
		return {
			isValid: false,
			error: `"${emoji}" is a reserved emoji for vault administration`,
			suggestion: "Pick an emoji besides 🥇, 🥈, or ⚛️"
		}
	}
    
    return { isValid: true };
}

/**
 * Creates a standardized file path based on note type and title
 * @param {string} noteType - Type of note (Primary, Secondary, Content)
 * @param {string} title - Note title
 * @returns {string} - Full file path
 */
function createDestinationPath(noteType, title) {
    switch(noteType) {
        case NOTE_TYPES.PRIMARY:
            return `${PATHS.PRIMARY_CATEGORIES}/${title}.md`;
        case NOTE_TYPES.SECONDARY:
            return `${PATHS.SECONDARY_CATEGORIES}/${title}.md`;
        case NOTE_TYPES.CONTENT:
            return `${PATHS.CONTENT}/${title}.md`;
        case NOTE_TYPES.CONTENT_TYPE:
	        return `${PATHS.CONTENT_TEMPLATES}/${title}`;
        default:
            throw new Error(`Unknown note type: ${noteType}`);
    }
}

//////////////////////////////////////////////////////////////////////////////////
//                           CORE WORKFLOW FUNCTIONS                           //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Prompts user to select note type with validation loop
 * @returns {Promise<string>} - Selected note type
 */
async function selectNoteType() {
    const options = [NOTE_TYPES.PRIMARY, NOTE_TYPES.SECONDARY, NOTE_TYPES.CONTENT];
    const displayOptions = [
        "🥇 Primary Category (High-level topics)",
        "🥈 Secondary Category (Mid-level topics)", 
        "⚛️ Content/Atomic Note (Specific content)"
    ];
    
    let selectedType = null;
    while (!selectedType) {
        selectedType = await tp.system.suggester(
            displayOptions, 
            options, 
            false,
            "Select note type (required):"
        );
        
        if (!selectedType) {
            showNotice("Note type selection is required to continue");
        }
    }
    
    return selectedType;
}

/**
 * Gets and validates note title with recursive prompting (5-count depth limit) for invalid titles
 * @param {string} noteType - Type of note being created ('Note', 'Primary Category', 'Secondary Category', or 'Content Type')
 * @returns {Promise<string>} - Validated title
 */
async function getValidatedNoteTitle(noteType) {
	let currentTitle = null;
    let attempts = 0;
    const maxAttempts = 5;
    
	while (attempts < maxAttempts) {
		attempts++;
		
		// Prompt for manual correction
        const promptText = attempts < maxAttempts 
            ? `Title for New ${noteType} (${maxAttempts - attempts + 1} attempts remaining):`
            : `Last chance - Title for New ${noteType}:`;

		currentTitle = await tp.system.prompt(promptText);

        const destinationPath = createDestinationPath(noteType, currentTitle);
        const validation = await isValidTitle(currentTitle, destinationPath, true);
        
        if (validation.isValid) {
            console.log(`Title "${currentTitle}" is valid for ${noteType}`);
            return currentTitle;
        }
        
        // Build helpful error message
        let errorMessage = `❌ ${validation.error}`;
        if (validation.suggestion) {
            errorMessage += `\n💡 Suggestion: ${validation.suggestion}`;
        }
        
        showNotice(errorMessage);
    }
    
    // If validation fails after max attempts, give user final choice
    showNotice("❌ Maximum validation attempts reached");
    
    const shouldContinue = await tp.system.suggester(
        ["🔄 Try one more time", "❌ Cancel note creation"],
        [true, false],
        false,
        "What would you like to do?"
    );
    
    if (!shouldContinue) {
        throw new Error("Note creation cancelled by user");
    }
    
    // One final attempt
    const finalTitle = await tp.system.prompt(
        "Final attempt - enter a valid title:"
    );
    
    // Quick final validation (don't loop again)
    const destinationPath = createDestinationPath(noteType, finalTitle);
    const finalValidation = await isValidTitle(finalTitle, destinationPath, true);
    
    if (finalValidation.isValid) {
        return finalTitle;
    } else {
        showNotice(`❌ Final validation failed: ${finalValidation.error}`);
        throw new Error("Could not create valid title after maximum attempts");
    }
}

//////////////////////////////////////////////////////////////////////////////////
//                            CATEGORY MANAGEMENT                              //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Retrieves all category files from a specific directory (no prefix removal needed)
 * @param {string} folderPath - Path to category folder
 * @returns {Promise<string[]>} - Array of category names
 */
async function getCategoriesFromFolder(folderPath) {
    const folder = app.vault.getAbstractFileByPath(folderPath);
    if (!folder || !folder.children) {
        return [];
    }
    
    return folder.children
        .filter(file => file.extension === "md")
        .map(file => file.basename)
        .sort();
}

/**
 * Gets all primary categories
 * @returns {Promise<string[]>} - Array of primary category names
 */
async function getPrimaryCategories() {
    return await getCategoriesFromFolder(PATHS.PRIMARY_CATEGORIES);
}

/**
 * Gets all secondary categories  
 * @returns {Promise<string[]>} - Array of secondary category names
 */
async function getSecondaryCategories() {
    return await getCategoriesFromFolder(PATHS.SECONDARY_CATEGORIES);
}

/**
 * Interactive category selection with multi-select capability
 * @param {string} categoryType - "primary" or "secondary"
 * @returns {Promise<string[]>} - Array of selected categories formatted as wiki links
 */
async function selectCategories(categoryType) {
    const isSecondary = categoryType === "secondary";
    const availableCategories = isSecondary ? await getSecondaryCategories() : await getPrimaryCategories();
    const categoryLabel = isSecondary ? "SECONDARY" : "PRIMARY";
    
    if (availableCategories.length === 0) {
        console.log(`No ${categoryType} categories found`);
        return [];
    }
    
    const selectedCategories = [];
    let continueSelecting = true;
    
    while (continueSelecting && selectedCategories.length < availableCategories.length) {
        const remainingCategories = availableCategories.filter(cat => !selectedCategories.includes(cat));
        
        if (remainingCategories.length === 0) break;
        
        const options = [...remainingCategories];
        const displayOptions = [...remainingCategories];
        
        // Add "Done" option after first selection
        if (selectedCategories.length > 0) {
            options.push("DONE");
            displayOptions.push("✅ Done (Finish Selection)");
        }
        
        const promptText = selectedCategories.length === 0 
            ? `Select ${categoryLabel} category to link back to:` 
            : `Selected: ${selectedCategories.join(", ")}. Select another or choose Done:`;
        
        const selection = await tp.system.suggester(displayOptions, options, false, promptText);
        
        if (selection === "DONE" || !selection) {
            continueSelecting = false;
        } else {
            selectedCategories.push(selection);
        }
    }
    
    // Format as proper wiki links (no prefixes needed)
    return selectedCategories.map(cat => `"[[${cat}]]"`);
}

//////////////////////////////////////////////////////////////////////////////////
//                              EMOJI MANAGEMENT                               //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Returns categorized emoji options for red team contexts
 * @returns {Object} - Object with emoji categories and options
 */
function getCategorizedEmojis() {
    return {
        "🔥 Attack & Exploitation": [
            {emoji: "💥", desc: "Attack/Impact"}, 
            {emoji: "💣", desc: "Payload/Exploit"},
            {emoji: "⚔️", desc: "Tool/Weapon"},
            {emoji: "🎯", desc: "Target/Precision"},
            {emoji: "🔨", desc: "Brute Force"},
            {emoji: "💉", desc: "Code Injection"},
            {emoji: "🎣", desc: "Phishing"},
            {emoji: "🎭", desc: "Social Engineering"},
            {emoji: "🐚", desc: "Shell/Command Line"},
            {emoji: "🔴", desc: "Red Team Activity"},
            {emoji: "🟣", desc: "Purple Team Activity"},
            {emoji: "🔓", desc: "Unlocked/Broken Access Control"},
            {emoji: "🕳️", desc: "Vulnerability/Security Risk"}
        ],
        "🛡️ Defense & Security": [
            {emoji: "🛡️", desc: "Defense/Protection"},
            {emoji: "🔒", desc: "Access Control"},
            {emoji: "🔐", desc: "Encryption"},
            {emoji: "🔑", desc: "Authentication"},
            {emoji: "🧱", desc: "Firewall/Blocking"},
            {emoji: "👁️", desc: "Monitoring/Detection"},
            {emoji: "⚠️", desc: "Warning/Alert"},
            {emoji: "🔵", desc: "Blue Team Activity"},
            {emoji: "🚨", desc: "Incident Response"},
            {emoji: "👣", desc: "IOC/Artifact"},
            {emoji: "🗝️", desc: "Old Key/Cryptography"}
        ],
        "📊 Analysis & Intelligence": [
            {emoji: "📊", desc: "Data Analysis"},
            {emoji: "🔍", desc: "Investigation/OSINT"},
            {emoji: "📈", desc: "Metrics/Reporting"},
            {emoji: "📋", desc: "Audit/Checklist"},
            {emoji: "📝", desc: "Documentation/Notes"},
            {emoji: "😈", desc: "Threat Actor/APT"},
            {emoji: "✍️", desc: "Reporting/Writing"},
            {emoji: "🎒", desc: "Education/Training"},
            {emoji: "👤", desc: "Person/Silhouette"},
            {emoji: "💡", desc: "Idea/Thought"},
            {emoji: "🗺️", desc: "Mind Map/Graph"},
            {emoji: "✅", desc: "Checklist/Playbook"},
            {emoji: "🎓", desc: "Student/Training"},
            {emoji: "📕", desc: "Records"}
        ],
        "🌐 Infrastructure": [
            {emoji: "🌐", desc: "Network/Web"},
            {emoji: "🧠", desc: "Artificial Intelligence"},
            {emoji: "📡", desc: "Communication/C2"},
            {emoji: "💻", desc: "Client/Endpoint"},
            {emoji: "🖥️", desc: "Server/Infrastructure"},
            {emoji: "📱", desc: "Mobile/Device"},
            {emoji: "🗄️", desc: "Database/Storage"},
            {emoji: "📶", desc: "Wireless/RF"},
            {emoji: "☁️", desc: "Cloud/Third-Party"},
            {emoji: "🏦", desc: "Vault/Secured Data"},
            {emoji: "🏗️", desc: "Infrastructure/Construction"},
            {emoji: "🧪", desc: "Lab Setup"}
        ],
        "⚙️ Technical": [
            {emoji: "🔧", desc: "Configuration/Tool"},
            {emoji: "🛠️", desc: "Toolkit/Development"},
            {emoji: "🤖", desc: "Automation/Script"},
            {emoji: "💾", desc: "Data/Persistence"},
            {emoji: "📦", desc: "Package/Bundle"},
            {emoji: "⚡", desc: "Performance/Speed"},
            {emoji: "💲", desc: "Command/Shell"},
            {emoji: "⚙️", desc: "Gear/Internals"}
        ]
    };
}

/**
 * Smart emoji selector with categorized options
 * @param {string} itemName - Name of item being tagged (for context)
 * @returns {Promise<string>} - Selected emoji
 */
async function selectEmoji(itemName) {
    const categories = getCategorizedEmojis();
    const options = [];
    const displayOptions = [];
    
    // Build categorized options
    for (const [categoryName, emojis] of Object.entries(categories)) {
        // Add category separator
        displayOptions.push(`--- ${categoryName} ---`);
        //options.push(`SEPARATOR_${categoryName}`);
        options.push(`${EMOJI_SELECTION.TYPES.SEPARATOR_PREFIX}${categoryName}`);
        
        // Add emoji options
        emojis.forEach(item => {
            displayOptions.push(`${item.emoji} ${item.desc}`);
            options.push(item.emoji);
        });
    }
    
    // Add utility options
    displayOptions.push(EMOJI_SELECTION.DISPLAY_TEXT.UTILITIES);
    options.push(`${EMOJI_SELECTION.TYPES.UTILITIES}Utilities`);
    displayOptions.push(EMOJI_SELECTION.DISPLAY_TEXT.MANUAL);
    options.push(EMOJI_SELECTION.TYPES.MANUAL);
    displayOptions.push(EMOJI_SELECTION.DISPLAY_TEXT.RANDOM);
    options.push(EMOJI_SELECTION.TYPES.RANDOM);
    
    const selection = await tp.system.suggester(
        displayOptions,
        options,
        false,
        `Select emoji for "${itemName}":`
    );

	// Handle user pressing escape key or exiting dropdown menu
    if (!selection) {
	    showNotice(`⚠️ Emoji selection cancelled by user, using default emoji "${EMOJI_SELECTION.DEFAULT_EMOJI}"`);
        return EMOJI_SELECTION.DEFAULT_EMOJI;
	}

	// Handle user selecting a category separator
	if (selection.startsWith(EMOJI_SELECTION.TYPES.UTILITIES)) {
	    showNotice(`⚠️ User selected a category header, using default emoji "${EMOJI_SELECTION.DEFAULT_EMOJI}"`);
	    return EMOJI_SELECTION.DEFAULT_EMOJI;
	}

	// Handle manual entry option
    if (selection === EMOJI_SELECTION.TYPES.MANUAL) {
        return await getValidatedEmoji();
    }

	// Handle randomized emoji choice option
    if (selection === EMOJI_SELECTION.TYPES.RANDOM) {
        const allEmojis = Object.values(categories).flat().map(e => e.emoji);
        return allEmojis[Math.floor(Math.random() * allEmojis.length)] || EMOJI_SELECTION.DEFAULT_EMOJI;
    }
    
    return selection;
}

/**
 * Gets and validates emoji with prompting for invalid entries
 * @returns {Promise<string>} - Validated emoji or fallback
 */
async function getValidatedEmoji() {
    let currentEmoji = null;
    let attempts = 0;
    const maxAttempts = 5;
    
    while (attempts < maxAttempts) {
        attempts++;
        
        // Prompt for emoji entry
        const promptText = attempts < maxAttempts
            ? `Search Tag Emoji for New Content Type (${maxAttempts - attempts + 1} attempts remaining):`
            : `Last chance - Emoji for New Content Type:`;

        currentEmoji = await tp.system.prompt(promptText);
        
        const validation = isValidEmoji(currentEmoji);
        
        if (validation.isValid) {
            console.log(`Emoji "${currentEmoji}" is valid for Content Type`);
            return currentEmoji;
        }
        
        // Build helpful error message
        let errorMessage = `❌ ${validation.error}`;
        if (validation.suggestion) {
            errorMessage += `\n💡 Suggestion: ${validation.suggestion}`;
        }
        
        showNotice(errorMessage);
        
        // Special handling for ZWJ emojis - allow user to proceed anyway
        if (validation.canProceedAnyway) {
            const proceed = await tp.system.suggester(
                ["✅ Use anyway (may cause display issues)", "❌ Try different emoji"],
                [true, false],
                false,
                `Continue with "${currentEmoji}"?`
            );
            
            if (proceed) {
                showNotice(`⚠️ Using complex emoji: ${currentEmoji}`);
                return currentEmoji;
            }
            // If the user chooses not to proceed, continue the loop
        }
    }
    
    // If validation fails after max attempts, give user final choice
    showNotice("❌ Maximum validation attempts reached");
    
    const shouldContinue = await tp.system.suggester(
        ["🔄 Try one more time", "📁 Use fallback emoji", "❌ Cancel creation"],
        ["retry", "fallback", "cancel"],
        false,
        "What would you like to do?"
    );
    
    if (shouldContinue === "cancel") {
        throw new Error("Emoji selection cancelled by user");
    }
    
    if (shouldContinue === "fallback") {
        showNotice("Using fallback emoji: 📁");
        return "📁";
    }
    
    // One final attempt
    const finalEmoji = await tp.system.prompt("Final attempt - enter a valid emoji:");
    
    const finalValidation = isValidEmoji(finalEmoji);
    
    if (finalValidation.isValid) {
        return finalEmoji;
    } else if (finalValidation.canProceedAnyway) {
        // Give one last chance for ZWJ override
        const finalProceed = await tp.system.suggester(
            ["✅ Use anyway", "📁 Use fallback emoji"],
            [true, false],
            false,
            `Final validation failed: ${finalValidation.error}. Use anyway?`
        );
        
        return finalProceed ? finalEmoji : "📁";
    } else {
        showNotice(`❌ Final validation failed: ${finalValidation.error}`);
        showNotice("Using fallback emoji: 📁");
        return "📁";
    }
}

//////////////////////////////////////////////////////////////////////////////////
//                           CONTENT TYPE MANAGEMENT                           //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Scans available content type templates and extracts their metadata
 * @returns {Promise<Object[]>} - Array of content type configurations
 */
async function getAvailableContentTypes() {
    const templatesFolder = app.vault.getAbstractFileByPath(PATHS.CONTENT_TEMPLATES);
    if (!templatesFolder || !templatesFolder.children) {
        return [];
    }
    
    const contentTypes = [];
    
    for (const folder of templatesFolder.children) {
        if (!folder.children) continue; // Skip non-folders
        
        const typeName = folder.name;
        
        // Extract emoji from metadata
        let emoji = EMOJI_SELECTION.DEFAULT_EMOJI; // Default fallback
        try {
            const metadataPath = `${PATHS.CONTENT_TEMPLATES}/${folder.name}/Metadata.md`;
            const metadataFile = app.vault.getAbstractFileByPath(metadataPath);
            
            if (metadataFile) {
                const frontmatter = app.metadataCache.getFileCache(metadataFile)?.frontmatter;
                
                if (frontmatter?.tags?.length > 0) {
                    // Find content type tag (exclude system tags)
                    const contentTag = frontmatter.tags.find(tag => 
                        typeof tag === 'string' &&
                        !tag.includes('Primary_Category') && 
                        !tag.includes('Secondary_Category')
                    );
                    
                    if (contentTag) {
                        const emojiMatch = contentTag.match(EMOJI_REGEX);
                        if (emojiMatch) {
                            emoji = emojiMatch[0];
                        }
                    }
                }
            }
        } catch (error) {
            logError(error, `Reading metadata for ${typeName}`, "default emoji");
        }
        
        contentTypes.push({
            name: typeName,
            emoji: emoji,
            displayName: `${emoji} ${typeName}`,
            searchTag: `${emoji}${typeName.replace(/\s+/g, '_')}`
        });
    }
    
    return contentTypes.sort((a, b) => a.name.localeCompare(b.name));
}

/**
 * Interactive content type selection or creation
 * @returns {Promise<Object>} - Selected/created content type configuration
 */
async function selectContentType() {
    const availableTypes = await getAvailableContentTypes();
    
    const options = [...availableTypes, "NEW_CONTENT_TYPE"];
    const displayOptions = [
        ...availableTypes.map(type => type.displayName),
        "➕ Create New Content Type"
    ];

    let selectedType = null;
    while (!selectedType) {
        selectedType = await tp.system.suggester(
            displayOptions,
            options,
            false,
            "Select Content Type or create new one:"
        );
        
        if (!selectedType) {
            showNotice("Content type selection is required");
        }
    }
    
    return selectedType === "NEW_CONTENT_TYPE" 
        ? await createNewContentType()
        : selectedType;
}

/**
 * Creates a new content type template structure
 * @returns {Promise<Object>} - New content type configuration
 */
async function createNewContentType() {
    // Get content type name
    let typeName = null;
    while (!typeName) {
        typeName = await getValidatedNoteTitle("Content Type");
    }
    
    // Select emoji
    const selectedEmoji = await selectEmoji();
    
    // Create template structure
    await createTemplateStructure(typeName, selectedEmoji);
    
    return {
        name: typeName,
        emoji: selectedEmoji,
        displayName: `${selectedEmoji} ${typeName}`,
        searchTag: `${selectedEmoji}${typeName.replace(/\s+/g, '_')}`
    };
}

/**
 * Creates template folder structure and files for new content type
 * @param {string} typeName - Content type name
 * @param {string} emoji - Selected emoji
 */
async function createTemplateStructure(typeName, emoji) {
    const basePath = `${PATHS.CONTENT_TEMPLATES}/${typeName}`;
    
    try {
        // Create folder
        await app.vault.createFolder(basePath);
        
        // Template files to create
        const templateFiles = [
            { source: "Metadata.md", target: "Metadata.md" },
            { source: "Footer.md", target: "Footer.md" },
            { source: "Body.md", target: "Body.md" }
        ];
        
        for (const fileInfo of templateFiles) {
            const sourceFilePath = `${PATHS.BASIC_TEMPLATE}/${fileInfo.source}`;
            const targetFilePath = `${basePath}/${fileInfo.target}`;
            
            const sourceFile = app.vault.getAbstractFileByPath(sourceFilePath);
            if (!sourceFile) {
                console.warn(`Source template file not found: ${sourceFilePath}`);
                continue;
            }
            
            let content = await app.vault.read(sourceFile);
            
            // Customize metadata file
            if (fileInfo.source === "Metadata.md") {
                content = content
                    .replace(
                        'tags:\n  - 📝Basic',
                        `tags:\n  - ${emoji}${typeName.replace(/\s+/g, '_')}`
                    )
                    .replace(
                        'type: Basic',
                        `type: ${typeName}`
                    );
            }
            
            // Create the file
            await app.vault.create(targetFilePath, content);
        }
        
        showNotice(`✅ Created new content type: ${emoji} ${typeName}`);
        
    } catch (error) {
        logError(error, "Creating template structure");
        throw error;
    }
}

//////////////////////////////////////////////////////////////////////////////////
//                              NOTE BUILDING                                  //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Creates a configuration object for note building
 * @param {Object} params - Parameters for note configuration
 * @returns {Object} - Note configuration object
 */
function createNoteConfig({
    noteType, 
    title, 
    primaryCategories = [], 
    secondaryCategories = [], 
    contentTypeConfig = null, 
    emoji = null
}) {
    const baseConfig = {
        noteType,
        title,
        primaryCategories,
        secondaryCategories
    };
    
    switch(noteType) {
        case NOTE_TYPES.PRIMARY:
            return {
                ...baseConfig,
                destination: `${PATHS.PRIMARY_CATEGORIES}/`,
                metadataTemplate: PATHS.PRIMARY_TEMPLATE_META,
                bodyTemplate: PATHS.PRIMARY_TEMPLATE_BODY,
                emoji
            };
            
        case NOTE_TYPES.SECONDARY:
            return {
                ...baseConfig,
                destination: `${PATHS.SECONDARY_CATEGORIES}/`,
                metadataTemplate: PATHS.SECONDARY_TEMPLATE_META,
                bodyTemplate: PATHS.SECONDARY_TEMPLATE_BODY
            };
            
        case NOTE_TYPES.CONTENT:
            return {
                ...baseConfig,
                destination: `${PATHS.CONTENT}/`,
                contentType: contentTypeConfig,
                metadataTemplate: `[[04 - Templates/04 - Content/${contentTypeConfig.name}/Metadata]]`,
                bodyTemplate: `[[04 - Templates/04 - Content/${contentTypeConfig.name}/Body]]`,
                footerTemplate: `[[04 - Templates/04 - Content/${contentTypeConfig.name}/Footer]]`
            };
            
        default:
            throw new Error(`Unsupported note type: ${noteType}`);
    }
}

/**
 * Loads all required templates for a note configuration
 * @param {Object} config - Note configuration object
 * @returns {Promise<Object>} - Template content object
 */
async function loadNoteTemplates(config) {
    const templates = {
        metadata: await tp.file.include(config.metadataTemplate),
        body: await tp.file.include(config.bodyTemplate)
    };
    
    if (config.footerTemplate) {
        templates.footer = await tp.file.include(config.footerTemplate);
    }
    
    return templates;
}

/**
 * Builds the complete note content from templates and configuration
 * @param {Object} config - Note configuration object
 * @returns {Promise<string>} - Complete note content
 */
async function buildNoteContent(config) {
    try {
		// Load and customize templates
        const templates = await loadNoteTemplates(config);

		// Build page header
        const pageTitle = `# [[${config.title}]]`;
		const header = templates.metadata + pageTitle
		
        // Each note has a metadata section and back-linked title
        const contentParts = [header];

		// Assemble content based on note type
		switch (config.noteType) {
            case NOTE_TYPES.CONTENT:
                contentParts.push(DIVIDER, templates.body, DIVIDER, templates.footer, DIVIDER, TIMESTAMP);
                break;
            case NOTE_TYPES.PRIMARY:
            case NOTE_TYPES.SECONDARY:
                contentParts.push(DIVIDER, templates.body, DIVIDER, TIMESTAMP);
                break;
            default:
                console.warn(`Unknown note type: ${config.noteType}`);
                contentParts.push(DIVIDER, templates.body, DIVIDER, TIMESTAMP);
        }
        
        return contentParts.join("\n");
        
    } catch (error) {
        logError(error, "Building note content");
        throw error;
    }
}

/**
 * Customizes metadata template based on note configuration
 * @param {string} metadata - Raw metadata template
 * @param {Object} config - Note configuration
 * @returns {string} - Customized metadata
 */
function customizeMetadata(metadata, config) {
    let customizedMetadata = metadata;
    
    switch(config.noteType) {
        case NOTE_TYPES.PRIMARY:
            if (config.emoji) {
                customizedMetadata = customizedMetadata.replace(
                    'tags:\n  - 🥇Primary_Category\n  - ADD_NEW_PRIMARY_CATEGORY_EMOJI',
                    `tags:\n  - 🥇Primary_Category\n  - ${config.emoji}${config.title.replace(/\s+/g, '_')}`
                );
            }
            break;
            
        case NOTE_TYPES.SECONDARY:
            if (config.primaryCategories.length > 0) {
                customizedMetadata = customizedMetadata.replace(
                    'primary categories:\n  - Add link(s) [[]] back to related PRIMARY categories',
                    `primary categories:\n  - ${config.primaryCategories.join('\n  - ')}`
                );
            }
            break;
            
        case NOTE_TYPES.CONTENT:
            // Update primary categories
            if (config.primaryCategories.length > 0) {
                customizedMetadata = customizedMetadata.replace(
                    'primary categories:\n  - Add link(s) [[]] back to related PRIMARY categories',
                    `primary categories:\n  - ${config.primaryCategories.join('\n  - ')}`
                );
            }
            
            // Update secondary categories
            if (config.secondaryCategories.length > 0) {
                customizedMetadata = customizedMetadata.replace(
                    'secondary categories:\n  - Add link(s) [[]] back to related SECONDARY categories',
                    `secondary categories:\n  - ${config.secondaryCategories.join('\n  - ')}`
                );
            }
            break;
    }
    
    return customizedMetadata;
}

//////////////////////////////////////////////////////////////////////////////////
//                                MAIN WORKFLOW                                //
//////////////////////////////////////////////////////////////////////////////////

/**
 * Main execution function - orchestrates the entire note creation workflow
 * @returns {string} - Personalized or fallback note content
 */
async function executeNoteCreation() {
    let config = null;
    
    try {
        console.log("=== Adversary Simulation Note Creation Started ===");
        
        // Step 1: Select note type
        const noteType = await selectNoteType();
        console.log(`Selected note type: ${noteType}`);
        
        // Step 2: Get and validate title
        const title = await getValidatedNoteTitle(noteType);
        console.log(`Validated title: "${title}"`);
        
        // Step 3: Execute type-specific workflow
        switch(noteType) {
            case NOTE_TYPES.PRIMARY:
                const emoji = await selectEmoji();
                config = createNoteConfig({ noteType, title, emoji });
                break;
                
            case NOTE_TYPES.SECONDARY:
                const primaryCategories = await selectCategories("primary");
                config = createNoteConfig({ noteType, title, primaryCategories });
                break;
                
            case NOTE_TYPES.CONTENT:
                const contentPrimaryCategories = await selectCategories("primary");
                const contentSecondaryCategories = await selectCategories("secondary");
                const contentType = await selectContentType();
                config = createNoteConfig({ 
                    noteType, 
                    title, 
                    primaryCategories: contentPrimaryCategories,
                    secondaryCategories: contentSecondaryCategories,
                    contentTypeConfig: contentType 
                });
                break;
                
            default:
                throw new Error(`Unsupported note type: ${noteType}`);
        }
        
        // Step 4: Move file to correct location
        await tp.file.move(config.destination + config.title);
        console.log(`File moved to: ${config.destination + config.title}`);
        
        // Step 5: Build and return note content
        const noteContent = await buildNoteContent(config);
        console.log("=== Note Creation Completed Successfully ===");
        
        return noteContent.trimEnd();
        
    } catch (error) {
        logError(error, "Note creation workflow");
        showNotice("❌ Note creation failed. Check console for details.");
        
        // Return minimal fallback content
        return `# ${tp.file.title || "New Note"}\n\n## Overview\n\n*Note creation encountered an error. Please try again or create manually.*\n\n---\n\n*Created Date*: <%+tp.file.creation_date(\"MMMM Do YYYY (HH:mm a)\")%\>  \n*Last Modified Date*: \<%+tp.file.last_modified_date(\"MMMM Do YYYY (HH:mm a)\")%\>`;
    }
}

//////////////////////////////////////////////////////////////////////////////////
//                                  EXECUTION                                  //
//////////////////////////////////////////////////////////////////////////////////

// Execute the main workflow and return the generated content
const generatedContent = await executeNoteCreation();

%><%* tR += `${generatedContent}` %>
