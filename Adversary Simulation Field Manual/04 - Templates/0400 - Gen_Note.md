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
    PRIMARY: "Primary",
    SECONDARY: "Secondary", 
    CONTENT: "Content"
};

const RESERVED_EMOJIS = new Set(['🥇', '🥈', '⚛️']);

const EMOJI_REGEX = /[\u{1F600}-\u{1F64F}]|[\u{1F300}-\u{1F5FF}]|[\u{1F680}-\u{1F6FF}]|[\u{1F1E0}-\u{1F1FF}]|[\u{2600}-\u{26FF}]|[\u{2700}-\u{27BF}]|[\u{1F900}-\u{1F9FF}]|[\u{1FA70}-\u{1FAFF}]|[\u{2300}-\u{23FF}]|[\u{2B50}]|[\u{2194}-\u{21AA}]|[\u{231A}-\u{231B}]|[\u{25AA}-\u{25FE}]/u;

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
 * Validates that a title is usable (not empty, not default, not duplicate)
 * @param {string} title - Title to validate
 * @param {string} destinationPath - Full path where note would be created
 * @param {boolean} isNote - Boolean indicating whether title is for a future note (true) or content type (false)
 * @returns {Promise<boolean>} - True if title is valid
 */
async function isValidTitle(title, destinationPath, isNote) {
    // Check for empty, undefined, or default titles
    if (!title || 
        title.includes('Untitled') || 
        title.trim() === "") {
        const message = isNote ? "Please provide a distinct note title" : "Please provide a distinct content type title";
        showNotice(message);
        return false;
    }
    
    // Check for duplicates
    const noteExists = await tp.file.exists(destinationPath);
    if (noteExists) {
		const message = isNote ? "A note with this title already exists in the destination directory" : "A content type with this title already exists in the destination directory";
        showNotice(message);
        return false;
    }
    
    return true;
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
        case "Content Type":
	        return `${PATHS.CONTENT_TEMPLATES}/${title}`;
        default:
            throw new Error(`Unknown note type: ${noteType}`);
    }
}

/**
 * Returns a sanitized file name
 * @param {string} title - Prospective note/content type title
 * @returns {string} - Sanitized note/content type title
*/
function sanitizeFileName(title) {
    return title
        .replace(/[/\\:*?"<>|]/g, '-')  // Replace invalid chars
        .replace(/^(CON|PRN|AUX|NUL|COM[1-9]|LPT[1-9])$/i, '$1_') // Windows reserved names
        .trim()
        .substring(0, 255); // Length limit
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
 * Gets and validates note title with recursive prompting for invalid titles
 * @param {string} initialTitle - Starting title (usually from tp.file.title)
 * @param {string} noteType - Type of note being created
 * @returns {Promise<string>} - Validated title
 */
async function getValidatedNoteTitle(initialTitle, noteType) {
    const destinationPath = createDestinationPath(noteType, initialTitle);
    if (await isValidTitle(initialTitle, destinationPath, true)) {
        console.log(`Title "${initialTitle}" is valid for ${noteType}`);
        return initialTitle;
    } 
    
    // Recursive prompting for new title
    const promptText = `Title for New ${noteType} ${noteType === NOTE_TYPES.CONTENT ? 'Note' : 'Category'}:`;
    const newTitle = await tp.system.prompt(promptText);
    
    return await getValidatedNoteTitle(newTitle, noteType);
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
 * Smart emoji selector with categorized options and availability checking
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
        options.push(`SEPARATOR_${categoryName}`);
        
        // Add emoji options
        emojis.forEach(item => {
            displayOptions.push(`${item.emoji} ${item.desc}`);
            options.push(item.emoji);
        });
    }
    
    // Add utility options
    displayOptions.push("--- Utilities ---");
    options.push("SEPARATOR_Utilities");
    displayOptions.push("✏️ Enter emoji manually");
    options.push("MANUAL_ENTRY");
    displayOptions.push("🎲 Random selection");
    options.push("RANDOM");
    
    const selection = await tp.system.suggester(
        displayOptions,
        options,
        false,
        `Select emoji for "${itemName}":`
    );
    
    // Handle special selections
    if (!selection || selection.startsWith("SEPARATOR_")) {
        showNotice("Using default emoji 📁");
        return "📁";
    }
    
    if (selection === "MANUAL_ENTRY") {
        return await handleManualEmojiEntry();
    }
    
    if (selection === "RANDOM") {
        const allEmojis = Object.values(categories).flat().map(e => e.emoji);
        return allEmojis[Math.floor(Math.random() * allEmojis.length)] || "📁";
    }
    
    return selection;
}

/**
 * Detects if an emoji contains Zero Width Joiner sequences that break Obsidian tags
 * @param {string} emoji - Emoji string to analyze
 * @returns {Object} - Analysis result
 */
function analyzeEmojiForObsidianTags(emoji) {
    if (!emoji || emoji.length === 0) {
        return {
            isTagSafe: false,
            reason: "Empty input"
        };
    }
    
    const codepoints = Array.from(emoji).map(char => char.codePointAt(0));
    const hasZWJ = codepoints.includes(0x200D); // Zero Width Joiner
    
    if (hasZWJ) {
        return {
            isTagSafe: false,
            reason: "Contains Zero Width Joiner (family/profession emoji)",
            codepoints: codepoints.map(cp => `U+${cp.toString(16).toUpperCase().padStart(4, '0')}`)
        };
    }
    
    return {
        isTagSafe: true,
        reason: "Should work fine in Obsidian tags"
    };
}

/**
 * Handles manual emoji entry with validation
 * @returns {Promise<string>} - Validated emoji or fallback
 */
async function handleManualEmojiEntry() {
    let attempts = 0;
    const maxAttempts = 3;
    
    while (attempts < maxAttempts) {
        const manualEmoji = await tp.system.prompt(
            `Enter emoji character (${attempts + 1}/${maxAttempts}):`
        );
        
        if (!manualEmoji) {
            showNotice("Manual entry cancelled");
            return "📁";
        }
        
        if (!EMOJI_REGEX.test(manualEmoji)) {
            attempts++;
            showNotice(`That doesn't appear to be a valid emoji. ${maxAttempts - attempts} attempts remaining.`);
            continue;
        }
        
        // Check specifically for ZWJ sequences
        const analysis = analyzeEmojiForObsidianTags(manualEmoji);
        
        if (!analysis.isTagSafe) {
            // Show detailed warning as a notice first
			showNotice(`⚠️ Complex emoji detected: "${manualEmoji}" contains Zero Width Joiner (ZWJ) sequences. Emojis with ZWJ sequences (e.g., 👨‍💻, 🕵️‍♂️) may cause tag display issues in Obsidian.`);

			const proceed = await tp.system.suggester(
			    ["✅ Use anyway", "❌ Try different emoji"],
			    [true, false],
			    false,
			    "Continue with this emoji?"
			);
            
            if (proceed) {
                showNotice(`⚠️ Using ZWJ emoji: ${manualEmoji}`);
                return manualEmoji;
            } else {
                attempts++;
                if (attempts < maxAttempts) {
                    showNotice(`Please try a different emoji. ${maxAttempts - attempts} attempts remaining.`);
                }
                continue;
            }
        }
        
        return manualEmoji;
    }
    
    showNotice("Max attempts reached. Using fallback emoji 📁");
    return "📁";
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
        let emoji = "📝"; // Default fallback
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
 * Gets and validates content type title with recursive prompting for invalid titles
 * @param {string} initialTitle - Starting title
 * @returns {Promise<string>} - Validated title
 */
async function getValidatedContentTypeTitle(initialTitle) {
    const destinationPath = createDestinationPath("Content Type", initialTitle);
    if (await isValidTitle(initialTitle, destinationPath, false)) {
        console.log(`Title "${initialTitle}" is valid for Content Type`);
        return initialTitle;
    } 
    
    // Recursive prompting for new title
    const promptText = `Title for New Content Type`;
    const newTitle = await tp.system.prompt(promptText);
    
    return await getValidatedContentTypeTitle(newTitle);
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
        typeName = await getValidatedContentTypeTitle(null);
    }
    
    // Select emoji
    const selectedEmoji = await selectEmoji(typeName);
    
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
            { source: "Header.md", target: "Header.md" },
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
                headerTemplate: `[[04 - Templates/04 - Content/${contentTypeConfig.name}/Header]]`,
                bodyTemplate: `[[04 - Templates/04 - Content/${contentTypeConfig.name}/Body]]`
            };
            
        default:
            throw new Error(`Unsupported note type: ${noteType}`);
    }
}

/**
 * Builds the complete note content from templates and configuration
 * @param {Object} config - Note configuration object
 * @returns {Promise<string>} - Complete note content
 */
async function buildNoteContent(config) {
    try {
        // Load and customize metadata template
        let metadata = await tp.file.include(config.metadataTemplate);
        metadata = customizeMetadata(metadata, config);
        
        // Build page title (no prefixes needed)
        const pageTitle = `# [[${config.title}]]`;
        
        // Load body template
        const body = await tp.file.include(config.bodyTemplate);
        
        // Add timestamps
        const timestamps = "\n*Created Date*: <%+tp.file.creation_date(\"MMMM Do YYYY (HH:mm a)\")%\>  \n*Last Modified Date*: \<%+tp.file.last_modified_date(\"MMMM Do YYYY (HH:mm a)\")%\>";
        
        // Assemble content based on note type
        const contentParts = [metadata, pageTitle];
        
        if (config.noteType === NOTE_TYPES.CONTENT) {
            const header = await tp.file.include(config.headerTemplate);
            contentParts.push(header);
        }
        
        contentParts.push(body, timestamps);
        
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
        console.log("=== Red Team Note Creation Started ===");
        
        // Step 1: Select note type
        const noteType = await selectNoteType();
        console.log(`Selected note type: ${noteType}`);
        
        // Step 2: Get and validate title
        const title = await getValidatedNoteTitle(tp.file.title, noteType);
        console.log(`Validated title: "${title}"`);
        
        // Step 3: Execute type-specific workflow
        switch(noteType) {
            case NOTE_TYPES.PRIMARY:
                const emoji = await selectEmoji(title);
                config = createNoteConfig({ noteType, title, emoji });
                break;
                
            case NOTE_TYPES.SECONDARY:
                const primaryCategories = await selectCategories("primary");
                config = createNoteConfig({ noteType, title, primaryCategories });
                break;
                
            case NOTE_TYPES.CONTENT:
                const contentType = await selectContentType();
                const contentPrimaryCategories = await selectCategories("primary");
                const contentSecondaryCategories = await selectCategories("secondary");
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
        
        return noteContent;
        
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
