### Todo

- [ ] Provide installation script (either in README or in a scripts directory)
  - [ ] Linux, Windows, macOS support (at the very least, a dedicated section in the README)
  - [ ] Automatically install, enable, configure required community plugins
- [ ] git support
  - [ ] .gitignore
  - [ ] Setup instructions added to README
- [ ] Testing script (necessary?)
- [ ] Add logic to pre-populate primary and secondary categories when creating new note
- [ ] Fix missing original notes
  - [ ] Dataview
  - [ ] Zettlekasten
- [ ] Fill out blank primary and secondary categories; only commit categories if they are back-linked by content notes
- [ ] Add Python script to scrape MITRE ATT&CK Framework and generate TTP notes

### In Progress

- [ ] Create example categories and content
  - [x] Basic
  - [ ] Tool (`impacket-GetUserSPNs`)
  - [x] TTP (`Steal or Forge Kerberos Tickets - Kerberoasting (T1558.003)`)
  - [x] Playbook (`PowerView Enumeration Checklist`)
  - [x] Mindmap (`AD - mindmap 2025 - 03`)
  - [x] Debrief (`Example Hack The Box Retired Machine`)
  - [x] Payload (`Reflective PowerShell Shellcode Runner`)
  - [ ] Infrastructure (`C2 Redirector`)
  - [x] Training Resources (`PEN-200`)

### Done ✓

- [x] Add and replace templates
- [x] Implement modified template script
- [x] Fix header bug where title becomes "Untitled" or "NaN"
- [x] Clean code writing for JS note generation script
- [x] Fix old categories and content to match new format(s)

