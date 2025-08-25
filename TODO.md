# Todo

## Not Started

- [ ] Modify templates to include a Dataview query section that pulls all notes backlinked to that category
  - [ ] Footer or body?

- [ ] Modify the "Obsidian - Getting Started" note to indicate that this fork was created by Anam0x and inspired by TrustedSec

- [ ] Add style guide to root of vault
  - [ ] Split up the style guide into atomic notes
  - [ ] What should the difference between links in the "Resources" table and footnotes be?

- [ ] Git support
  - [ ] .gitignore
  - [ ] README
  - [ ] Update Obsidian Git note
  - [ ] Setup scripts
   - [ ] Bash for *NIX-like
   - [ ] PowerShell for Windows

## In Progress

### Highest Priority (Blocking Issues)

- [ ] Critical error handling
  - [ ] Sanitize file names
  - [ ] Infinite recursion in `getValidatedNoteTitle` and `getValidatedContentTypeTitle` (these two functions could probably be merged)
  - [ ] Templater plugin dependency (check if available first)
  - [ ] Validate template content before processing
  - [ ] Array bounds issues with `allEmojis` constant
  - [ ] Potentially reading stale metadata in `getAvailableContentTypes`

### High Priority (Core Functionality and Usability)

- [ ] Provide template for new content types (e.g., "Command", "Idea", "IOC")
  - [ ] Replace all blockquote template instances with admonition block-styled content
- [ ] Header/Footer template restructuring (impacts all new notes)
  - [ ] Replace "Header" with "Footer"
  - [ ] Create a utility function to add a dividing line between individual template structures
  - [ ] Remove the "Resources" table and footnotes from all content templates and move them to the new "Footer" element
  - [ ] Update main script
- [ ] Installation instructions (enables adoption)
  - [ ] README
  - [ ] Rewrite the "Vault Appendix - Modifying Vault Structure" note
  - [ ] Linux, Windows, macOS, Android, iOS support
  - [ ] Automatically install, enable, configure required community plugins (if possible)

### Medium Priority (Enhancement)

- [ ] Add Python script to scrape MITRE ATT&CK Framework and generate TTP notes
- [ ] Personal/unfinished ideas directory
- [ ] Example notes for new content types (e.g., Command, Idea, IOC)
  - [ ] Consider adding example notes for each new content type

### Low Priority (Cleanup/Polish)

- [ ] Break up large notes (PowerView, Zettelkasten) to maintain principle of atomicity
- [ ] Race condition handling (low probability for personal use)

## Done ✓

- [x] Emoji issues
  - [x] Fix emoji detection/regex issues
  - [x] Obsidian does not like Zero Width Joiner (U+200D) sequence emojis as tags; implement logic to warn users before attempting to assign ZWJ emoji as search tag

- [x] Removed unnecessary prefixes from notes
  - [x] Renaming links

- [x] Add more scripting logic
  - [x] Prompt to create new search tag for all new primary categories
  - [x] Prompt to pre-populate metadata links in new secondary categories with primary categories
  - [x] Prompt to pre-populate metadata links in new content notes with primary and secondary categories
  - [x] Fixed issue where empty files were being created if operation was cancelled by forcing each step to loop until a valid option is chosen; this comes with the tradeoff of limiting user agency and becoming "trapped" in a workflow, but is acceptable
  - [x] Adding and removing content types (e.g., "TTP", "Tool", "Basic")
  - [x] Clean code

- [x] Rename "Debrief" to "Case Study" (a "case study" covers both personal reflection post-engagement and formal analysis of publicly referenceable attacks/incidents)

- [x] Content type ideas
  - [x] Idea (bulb)
  - [x] Biography (silhouette)
  - [x] Vulnerability (hole)
  - [x] IOC (footprints)
  - [x] Lab Setup (test tube)
  - [x] Command (dollar sign)
  - [x] Artificial Intelligence (brain)

- [x] Primary category ideas
  - [x] Social Engineering (theater mask emoji as the special search tag)
  - [x] OS Internals (gear)
  - [x] Detection Engineering (shield)
  - [x] Threat Intelligence (smiling imp with horns)
  - [x] OSINT (magnifying glass)
  - [x] Cryptography (old key)

- [x] Add "special tags" for primary category titles; secondary categories should not be necessary
  - [x] Change currently tracked notes
  - [x] Confirmed that future content/primary category emojis are supported tag options
  - [x] Update templates to reflect changes

- [x] Fix missing original notes
  - [x] Dataview
  - [x] Emoji Toolbar
  - [x] Zettlekasten

- [x] Create example categories and content
  - [x] Basic
  - [x] Tool (`impacket-GetUserSPNs`)
  - [x] TTP (`Steal or Forge Kerberos Tickets - Kerberoasting (T1558.003)`)
  - [x] Playbook (`PowerView Enumeration Checklist`)
  - [x] Mindmap (`AD - mindmap 2025 - 03`)
  - [x] Debrief (`Example Hack The Box Retired Machine`)
  - [x] Payload (`Reflective PowerShell Shellcode Runner`)
  - [x] Infrastructure (`C2 Redirector`)
  - [x] Training Resources (`PEN-200`)
- [x] Add and replace templates
- [x] Implement modified template script
- [x] Fix header bug where title becomes "Untitled" or "NaN"
- [x] Clean code writing for JS note generation script
- [x] Fix old categories and content to match new format(s)

