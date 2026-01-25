# CLAUDE.md - AI Assistant Guide for Awesome-Hacking

This document provides guidance for AI assistants working with this repository.

## Repository Overview

**Awesome-Hacking** is a curated collection of awesome lists for hackers, pentesters, and security researchers. It serves as a meta-list that aggregates links to other specialized security-related "awesome" repositories on GitHub.

- **Organization**: [Hack-with-Github](https://github.com/Hack-with-Github)
- **License**: CC0 1.0 Universal (Public Domain)
- **Purpose**: Aggregate and organize security/hacking learning resources

## Repository Structure

```
Awesome-Hacking/
├── README.md           # Main content - curated list of repositories
├── contributing.md     # Contribution guidelines
├── LICENSE             # CC0 1.0 Universal license
├── awesome_hacking.jpg # Repository banner image
└── .github/
    └── workflows/
        └── lock-threads.yml  # Auto-locks inactive issues/PRs after 7 days
```

## Content Organization

The README.md contains two main tables:

1. **Awesome Repositories** - Primary security/hacking resource lists covering:
   - Penetration testing (pentest, red teaming, exploit development)
   - Security domains (web, mobile, IoT, vehicle, mainframe)
   - Defensive security (incident response, threat intelligence, OSINT)
   - Specialized topics (CTF, malware analysis, fuzzing, YARA)

2. **Other Useful Repositories** - Supporting resources including:
   - Cheatsheets and reference materials
   - Vulnerable environments for practice
   - Payloads and wordlists
   - Machine learning for security

## Key Conventions

### Table Format
All entries follow this markdown table format:
```markdown
Repository | Description
---- | ----
[Repository Name](URL) | Brief description
```

### Alphabetical Ordering
Entries within each table section MUST be in alphabetical order by repository name.

### Entry Guidelines
- Repository names should use the display name from the linked repo
- Descriptions should be concise (one sentence)
- URLs must point to valid GitHub repositories or tools
- No trailing whitespace

## Contribution Workflow

1. Fork the repository
2. Add new entry to `README.md` in the appropriate section
3. Ensure alphabetical ordering is maintained
4. Submit a pull request

### What Gets Added
- Links to curated "awesome lists" related to security/hacking
- Links to significant security tools and resources
- Resources must be actively maintained and high quality

### What Gets Removed
- Broken links (report via issue)
- Abandoned or low-quality resources

## Automation

### Lock Threads Workflow
- Runs hourly via cron
- Auto-locks issues and PRs after 7 days of inactivity
- Prevents stale discussions on merged/closed items

## AI Assistant Guidelines

### DO
- Help users find relevant security resources from the list
- Suggest appropriate categories for new resources
- Verify alphabetical ordering when adding entries
- Check that URLs are properly formatted
- Maintain consistent table formatting

### DON'T
- Add resources without verifying they exist and are active
- Break alphabetical ordering in tables
- Remove entries without clear justification (broken link, etc.)
- Modify the table format or structure
- Add duplicate entries

### When Adding New Entries
1. Verify the resource URL is valid
2. Determine the correct table (Awesome Repositories vs Other Useful)
3. Find the correct alphabetical position
4. Match existing description style (brief, no period at end typically)
5. Ensure markdown table alignment is preserved

### Security Context
This repository focuses on **ethical hacking and security research**. All resources are intended for:
- Authorized penetration testing
- Security research and education
- CTF competitions
- Defensive security improvements

AI assistants should maintain this ethical context when discussing or extending the content.
