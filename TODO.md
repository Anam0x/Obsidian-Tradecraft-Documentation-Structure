# Todo

## Not Started

- [ ] Bugs
  - [ ] Unable to invoke Templater script by clicking the "New Note" icon (currently only works by right-clicking directory and selecting "New Note" option)
    - [ ] Would have to set Templater to run at the root of the vault; unsure if this is the desired behavior
  - [ ] Some emojis (e.g., m/f detective) are not valid characters for tags
  - [ ] Debrief and lab setup emojis not being detected

- [ ] Figure out what to do with seemingly obsolete "Header" template structure element

- [ ] Add designated section for unfinished ideas (reintroduce the "Personal" directory?)

- [ ] Add Python script to scrape MITRE ATT&CK Framework and generate TTP notes

## In Progress

### Highest Priority

- [ ] Add more scripting logic
  - [x] Prompt to create new search tag for all new primary categories
  - [x] Prompt to pre-populate metadata links in new secondary categories with primary categories
  - [x] Prompt to pre-populate metadata links in new content notes with primary and secondary categories
  - [x] Fixed issue where empty files were being created if operation was cancelled by forcing each step to loop until a valid option is chosen; this comes with the tradeoff of limiting user agency and becoming "trapped" in a workflow, but is acceptable
  - [x] Adding and removing content types (e.g., "TTP", "Tool", "Basic")
  - [ ] Error/interrupted execution flow handling
  - [ ] Clean code

### Medium Priority

- [ ] git support
  - [ ] .gitignore
  - [ ] Setup instructions added to README

### Lowest Priority

- [ ] Provide installation instructions (either in README or in a scripts directory)
  - [ ] Content page for modifying vault structure (i.e., primary/secondary categories, content types, templates)
    - [ ] Instructions
    - [ ] GIFs
  - [ ] Linux, Windows, macOS support (at the very least, a dedicated section in the README)
  - [ ] Automatically install, enable, configure required community plugins

- [ ] Consider replacing `PowerView Enumeration Checklist` with a single `PowerView - Enumerate Users` note to maintain principle of atomicity

- [ ] Add style guide to root of vault
  - [ ] Split up the style guide into atomic notes
  - [ ] What should the difference between links in the "Resources" table and footnotes be?

- [ ] Break up the "Zettelkasten" note into multiple atomic notes
  - [ ] Biography content type for Niklas Luhmann
  - [ ] Idea content type for atomic notes
  - [ ] Consider other methods for reducing the note

## Done ✓

- [x] Rename "Debrief" to "Case Study" and optionally change emoji (a "case study" covers both personal reflection post-engagement and formal analysis of publicly referenceable attacks/incidents)

- [x] Content type ideas
  - [x] Idea (bulb)
  - [x] Biography (silhouette)
  - [x] Vulnerability (hole)
  - [x] IOC (footprints)
  - [x] Lab Setup (test tube)
  - [x] Command (dollar sign)

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

