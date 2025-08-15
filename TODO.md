# Todo

## Not Started

- [ ] Figure out what to do with seemingly obsolete "Header" template structure element

- [ ] Add designated section for unfinished ideas (reintroduce the "Personal" directory?)

- [ ] Add Python script to scrape MITRE ATT&CK Framework and generate TTP notes

- [ ] Primary category ideas
  - [ ] Social Engineering (theater mask emoji as the special search tag)
  - [ ] OS Internals (gear)
  - [ ] Purple Team (purple circle)
  - [ ] Detection Engineering (blue circle)
  - [ ] Threat Intelligence (smiling imp with horns)
  - [ ] Mobile Security (cellphone)
  - [ ] OSINT (detective)
  - [ ] Cryptography (secured lock with key)

- [ ] Content type ideas
  - [ ] Idea (bulb)
  - [ ] Biography (silhouette)
  - [ ] Vulnerability (hole)
  - [ ] IOC (red alert)
  - [ ] Lab Setup (test tube)
  - [ ] Command (dollar sign)

## In Progress

### Highest Priority

- [ ] Add "special tags" for primary category titles; secondary categories should not be necessary
  - [ ] Update "Obsidian - Modifying Vault Structure" note to reflect changes

- [ ] Rename "Debrief" to "Case Study" and optionally change emoji (a "case study" covers both personal reflection post-engagement and formal analysis of publicly referenceable attacks/incidents)

### Medium Priority

- [ ] Add more scripting logic
  - [ ] Prompt to pre-populate tags and aliases for all new categories and content notes
  - [ ] Prompt to pre-populate metadata links in new secondary categories with primary categories
  - [ ] Prompt to pre-populate metadata links in new content notes with primary and secondary categories
  - [ ] Adding and removing content types (e.g., "TTP", "Tool", "Basic")

- [ ] git support
  - [ ] .gitignore
  - [ ] Setup instructions added to README

### Lowest Priority

- [ ] Provide installation instructions (either in README or in a scripts directory)
  - [ ] Content page for modifying vault structure (i.e., primary/secondary categories, content types, templates)
    - [x] Instructions
    - [ ] GIFs
  - [ ] Linux, Windows, macOS support (at the very least, a dedicated section in the README)
  - [ ] Automatically install, enable, configure required community plugins

- [ ] Consider replacing `PowerView Enumeration Checklist` with a single `PowerView - Enumerate Users` note to maintain principle of atomicity

- [ ] Add style guide to root of vault
  - [ ] Split up the style guide into atomic notes
  - [ ] What should the difference between links in the "Resources" table and footnotes be?

## Done ✓

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

