### Todo

- [ ] Add "special tags" for primary category titles; secondary categories should not be necessary
  - [ ] Update "Obsidian - Modifying Vault Structure" note to reflect changes
- [ ] git support
  - [ ] .gitignore
  - [ ] Setup instructions added to README
- [ ] Add Python script to scrape MITRE ATT&CK Framework and generate TTP notes
- [ ] Add more scripting logic
  - [ ] Prompt to pre-populate tags and aliases for all new categories and content notes
  - [ ] Prompt to pre-populate metadata links in new secondary categories with primary categories
  - [ ] Prompt to pre-populate metadata links in new content notes with primary and secondary categories
  - [ ] Adding and removing content types (e.g., "TTP", "Tool", "Basic")
- [ ] Primary category ideas
  - [ ] Social Engineering

### In Progress

- [ ] Fix missing original notes
  - [x] Dataview
  - [ ] Emoji Toolbar
  - [ ] Zettlekasten
- [ ] Add style guide to root of vault
- [ ] Provide installation instructions (either in README or in a scripts directory)
  - [ ] Content page for modifying vault structure (i.e., primary/secondary categories, content types, templates)
    - [x] Instructions
    - [ ] GIFs
  - [ ] Linux, Windows, macOS support (at the very least, a dedicated section in the README)
  - [ ] Automatically install, enable, configure required community plugins

### Done ✓

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

