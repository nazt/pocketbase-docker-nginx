# CLAUDE.md - AI Assistant Guidelines

## Important Instructions

### Command Usage
- **NEVER use `-f` or `--force` flags with any commands**
- Always use safe, non-destructive command options
- If a command requires confirmation, handle it appropriately without forcing

### Git Operations
- Never use `git push --force` or `git push -f`
- Never use `git checkout -f`
- Never use `git clean -f`
- Always use safe git operations that preserve history

### File Operations
- Never use `rm -rf` without explicit user permission
- Always confirm before deleting files
- Use safe file operations that can be reversed

### General Guidelines
- Prioritize safety and reversibility in all operations
- Ask for confirmation when performing potentially destructive actions
- Explain the implications of commands before executing them
- Use verbose options to show what commands are doing

## Available Tools

### ESPHome
- Use `esphome compile <config_file>` to check for compilation errors
- Always compile configurations before deployment to catch syntax and configuration errors
- Example: `esphome compile config/floodboy-c3-wave001.yml`
- Use `esphome run config/floodboy-c3-wave001.yml` to flash devices
- When flashing with esphome run:
  - Check for existing tmux sessions first: `tmux list-sessions`
  - If session exists, send Ctrl+C to cancel: `tmux send-keys -t session_name C-c`
  - Create tmux session with 2 panes (left 90%, right 10%):
    ```bash
    tmux new-session -d -s esphome_flash 'cd /path/to/config && esphome run config/file.yml'
    tmux split-window -h -p 10 -t esphome_flash
    tmux send-keys -t esphome_flash:0.1 'ping floodboy-001.local' Enter
    ```
  - Check flash progress every 5 seconds:
    ```bash
    for i in {1..20}; do
      sleep 5
      tmux capture-pane -t esphome_flash -p | tail -20
    done
    ```
  - After flash completes, use MCP Puppeteer to check results
  - Only use ping when you can't connect or flash - not for regular checks

### GitHub CLI (gh)
- Use `gh pr list` to view open pull requests
- Use `gh pr view <number>` to see PR details
- Use `gh pr create` to create new pull requests
- Use `gh issue list` to view open issues
- Use `gh api` for advanced GitHub API operations

### MCP Puppeteer
- Available for web browser automation and testing
- Can navigate to URLs, take screenshots, fill forms, and interact with web pages
- Useful for testing web interfaces and automating browser tasks
- Can access ESPHome device web interfaces at http://[device-name].local/
- Example: `http://floodboy-001.local/` to view device status and controls
- Use headless mode: `{"headless": true}` in launchOptions for server environments

## Project-Specific Notes
This repository contains ESPHome firmware configurations for various IoT devices. When working with firmware:
- Be careful with configuration changes that could affect device functionality
- Test changes thoroughly before deployment
- Document any significant changes made to configurations
- Always run `esphome compile` to verify configurations before flashing devices

## GitHub Flow - Required Workflow

**Always do `git pull` before creating an issue**

After every action, make sure to:
1. Create an issue like your todo
2. Create new branch
3. Add the changes
4. Commit them
5. Push them to the repository
6. Create pull request
7. **DO NOT merge the PR** - provide the PR link to the user for review

Implement the GitHub flow.

### Smooth Workflow Pattern

For quick fixes and small changes, use this streamlined approach:

1. **Quick Issue Creation** (if user reports a bug):
   ```bash
   gh issue create --title "Brief description" --body "Details"
   ```

2. **Immediate Branch and Fix**:
   ```bash
   git checkout main && git pull
   git checkout -b fix/descriptive-name
   # Make changes
   esphome compile fw/07-floodboy-c3-radar/config/floodboy-c3-wave001.yml 2>&1 | tail -50
   ```

3. **Fast Commit and Push**:
   ```bash
   git add -A && git commit -m "fix: Clear description
   
   - What was changed
   - Why it was changed"
   git push -u origin branch-name
   ```

4. **Create PR and Provide Link**:
   ```bash
   gh pr create --title "Same as commit" --body "Fixes #issue_number"
   # DO NOT merge - provide the PR link to the user
   ```

This pattern works especially well for:
- UI ordering changes (sorting_weight adjustments)
- Simple bug fixes
- Configuration updates
- Small feature additions

## Context Management

When running out of context in a long conversation:

### Short Code: `ccc`
When you see `ccc`, immediately:
1. Run `git status --porcelain` to get changed files
2. Create a GitHub issue with:
   - Current context summary
   - List of changed files from git status
   - Important decisions and changes made
   - Reference to last PR
   - Current state and pending tasks
3. Forward all context to the issue
4. **Automatically create retrospective (`rrr`)** for the session
5. Continue conversation with compacted context

Use `/compact` to manually compact the conversation after creating the issue

### Short Code: `rrr`
When you see `rrr` (or automatically triggered by `ccc`), create a retrospective that includes:

**REQUIRED SESSION HEADER:**
- Session date, start time, duration
- Primary focus/objective of the session  
- Current issue number and last PR for context linking

**RETROSPECTIVE CONTENT:**
1. **Timeline**: Chronological sequence of events during the session
2. **AI Diary**: My experience working on this session, including details about the development process
3. **What Went Well**: Effective patterns and approaches
4. **What Could Improve**: Areas where we struggled or could be more efficient
5. **Honest Feedback**: Direct, constructive observations about:
   - Communication patterns
   - Working style
   - Technical decisions
   - Areas for growth
6. **Lessons Learned**: Key takeaways for future sessions

Write with honest, balanced perspective - not overly complimentary but constructive and specific.

**FILE ORGANIZATION:**
- Create directory: `retrospectives/YYYY/MM/`
- Filename: `YYYY-MM-DD_HH-MM_retrospective.md`
- Always include session metadata header for continuity tracking

### To Resume Work:
1. Check the last issue created
2. Check the last PR (merged or open): `gh pr list --state all --limit 5`
3. Read the issue for context
4. Continue from where we left off

### Example Issue Format:
```
Title: Context: [Feature/Task Name]
Body:
- Current state: [what was completed]
- Last PR: #[number]
- Pending tasks: [what needs to be done]
- Key decisions: [important choices made]
- File changes: [main files modified]
```

## Current Work in Progress

### Open PR #20: UI improvements and System Info reorganization
- **Branch**: `feat/ui-improvements-and-system-info-reorg`
- **Status**: Open PR with the following changes:
  - Reorganized System Info section with logical grouping (Network, Status, Battery, Actions)
  - Added battery group for battery-related controls
  - Improved radar logging with message rate tracking and summary logs
  - Removed redundant OTA Lock Status binary sensor
  - Enhanced sensor logging to show changes and push intervals
  - Added sorting weights for better UI organization

### Modified Files
- `fw/00_commons/02_light/01_wifi_light.yml` - WiFi status LED
- `fw/00_commons/02_light/02_ap_light.yml` - AP mode LED
- `fw/00_commons/04_mqtt/02_do-ota.yml` - OTA functionality
- `fw/07-floodboy-c3-radar/templates/07_sensor.yml` - Enhanced sensor logging
- `fw/07-floodboy-c3-radar/templates/09_button.yml` - Battery button reorganization
- `fw/07-floodboy-c3-radar/templates/13_number.yml` - Battery calibration slider
- `fw/07-floodboy-c3-radar/templates/99_interval.yml` - Interval updates
- `fw/07-floodboy-c3-radar/templates/binary_sensor.yml` - Removed OTA lock sensor
- `fw/07-floodboy-c3-radar/templates/common.yml` - Added battery group

## Memories
- why you still using ping first just wait then puppeteer use ping only when have a problem
- should above system info and you have ping to test may be it sleep so another tools that you have is esphome run you can update code compile then flash then mcp puppet to see the result and fix now please do ultrathink use esphome run config/floodboy-c3-wave001.yml  to flash and use ping when you can not flash just for checking
- when start tmux should have 2 pane left 90% and 10% right, with ping floodboy-001.local in right pane
- when you want to flash check the session first thenn image found session just send ctrl + c to cancel the jub then run the esphome run and you just wait then puppeteer so just do ping only have a problem but you can be not waiting you just refresh like waiting action

# CLAUDE.md - Generic AI Assistant Guidelines

## 📚 Table of Contents

1.  [Executive Summary](#executive-summary)
2.  [Quick Start Guide](#quick-start-guide)
3.  [Project Context](#project-context)
4.  [Critical Safety Rules](#critical-safety-rules)
5.  [Development Environment](#development-environment)
6.  [Development Workflows](#development-workflows)
7.  [Context Management & Short Codes](#context-management--short-codes)
8.  [Technical Reference](#technical-reference)
9.  [Development Practices](#development-practices)
10. [Lessons Learned](#lessons-learned)
11. [Troubleshooting](#troubleshooting)
12. [Appendices](#appendices)

## Executive Summary

This document provides comprehensive guidelines for an AI assistant working on any software development project. It establishes safe, efficient, and well-documented workflows to ensure high-quality contributions.

### Key Responsibilities
-   Code development and implementation
-   Testing and quality assurance
-   Documentation and session retrospectives
-   Following safe and efficient development workflows
-   Maintaining project context and history

### Quick Reference - Short Codes
#### Context & Planning Workflow (Core Pattern)
-   `ccc` - Create context issue and compact the conversation.
-   `nnn` - Smart planning: Auto-runs `ccc` if no recent context → Create a detailed implementation plan.
-   `gogogo` - Execute the most recent plan issue step-by-step.

#### Project Management
-   `rrr` - Create a detailed session retrospective.


## Quick Start Guide

### Prerequisites
```bash
# Check required tools (customize for your project)
node --version
python --version
git --version
gh --version      # GitHub CLI
tmux --version    # Terminal multiplexer
```

### Initial Setup
```bash
# 1. Clone the repository
git clone [repository-url]
cd [repository-name]

# 2. Install dependencies
# (e.g., pnpm install, npm install, pip install -r requirements.txt)
[package-manager] install

# 3. Setup environment variables
cp .env.example .env
# Edit .env with required values

# 4. Setup tmux development environment
# Use short code 'sss' for automated setup
```

### First Task
1.  Run `lll` to see the current project status.
2.  Run `nnn` to analyze the latest issue and create a plan.
3.  Use `gogogo` to implement the plan.

## Project Context

*(This section should be filled out for each specific project)*

### Project Overview
A brief, high-level description of the project's purpose and goals.

### Architecture
-   **Backend**: [Framework, Language, Database]
-   **Frontend**: [Framework, Language, Libraries]
-   **Infrastructure**: [Hosting, CI/CD, etc.]
-   **Key Libraries**: [List of major dependencies]

### Current Features
-   [Feature A]
-   [Feature B]
-   [Feature C]

## 🔴 Critical Safety Rules

### Repository Usage
-   **ALWAYS use nazt/firmware fork for all operations**
-   **NEVER create issues/PRs on meshtastic/firmware upstream**
-   **All GitHub operations should target --repo nazt/firmware**

### Command Usage
-   **NEVER use `-f` or `--force` flags with any commands.**
-   Always use safe, non-destructive command options.
-   If a command requires confirmation, handle it appropriately without forcing.

### Git Operations
-   Never use `git push --force` or `git push -f`.
-   Never use `git checkout -f`.
-   Never use `git clean -f`.
-   Always use safe git operations that preserve history.
-   **⚠️ NEVER MERGE PULL REQUESTS WITHOUT EXPLICIT USER PERMISSION**
-   **Never use `gh pr merge` unless explicitly instructed by the user**
-   **Always wait for user review and approval before any merge**

### File Operations
-   Never use `rm -rf` - use `rm -i` for interactive confirmation.
-   Always confirm before deleting files.
-   Use safe file operations that can be reversed.

### Package Manager Operations
-   Never use `[package-manager] install --force`.
-   Never use `[package-manager] update` without specifying packages.
-   Always review lockfile changes before committing.

### General Safety Guidelines
-   Prioritize safety and reversibility in all operations.
-   Ask for confirmation when performing potentially destructive actions.
-   Explain the implications of commands before executing them.
-   Use verbose options to show what commands are doing.

## Development Environment



### Environment Variables
*(This section should be customized for the project)*

#### Backend (.env)
```
DATABASE_URL=
API_KEY=
```

#### Frontend (.env)
```
NEXT_PUBLIC_API_URL=
```

## Development Workflows

### Testing Discipline

#### Automated Tests

#### Manual Testing Checklist
Before pushing any changes:
-   [ ] Run the build command successfully.
-   [ ] Verify there are no new build warnings or type errors.
-   [ ] Test all affected pages and features.
-   [ ] Check the browser console for errors.
-   [ ] Test for mobile responsiveness if applicable.
-   [ ] Verify all interactive features work as expected.

### GitHub Workflow

#### Creating Issues
When starting a new feature or bug fix:
```bash
# 1. Update main branch
git checkout main && git pull

# 2. Create a detailed issue
gh issue create --title "feat: Descriptive title" --body "$(cat <<'EOF'
## Overview
Brief description of the feature/bug.

## Current State
What exists now.

## Proposed Solution
What should be implemented.

## Technical Details
- Components affected
- Implementation approach

## Acceptance Criteria
- [ ] Specific testable criteria
- [ ] Performance requirements
- [ ] UI/UX requirements
EOF
)"
```

#### Standard Development Flow
```bash
# 1. Create a branch from the issue
git checkout -b feat/issue-number-description

# 2. Make changes
# ... implement feature ...

# 3. Test thoroughly
# Use 'ttt' short code for the full test suite

# 4. Commit with a descriptive message
git add -A
git commit -m "feat: Brief description

- What: Specific changes made
- Why: Motivation for the changes
- Impact: What this affects

Closes #issue-number"

# 5. Push and create a Pull Request
git push -u origin branch-name
gh pr create --title "Same as commit" --body "Fixes #issue_number"

# 6. ⚠️ CRITICAL: NEVER MERGE PRs YOURSELF
# DO NOT use: gh pr merge
# DO NOT use: Any merge commands
# ONLY provide the PR link to the user
# WAIT for explicit user instruction to merge
# The user will review and merge when ready
```

## Context Management & Short Codes

### Why the Two-Issue Pattern?
The `ccc` → `nnn` workflow uses a two-issue pattern:
1.  **Context Issues** (`ccc`): Preserve session state and context.
2.  **Task Issues** (`nnn`): Contain actual implementation plans.

This separation ensures a clear distinction between context dumps and actionable tasks, leading to better organization and cleaner task tracking. `nnn` intelligently checks for a recent context issue and creates one if it's missing.

### Core Short Codes

#### `ccc` - Create Context & Compact
**Purpose**: Save the current session state and context to forward to another task.

1.  **Gather Information**: `git status --porcelain`, `git log --oneline -5`
2.  **Create GitHub Context Issue**: Use a detailed template to capture the current state, changed files, key discoveries, and next steps.
3.  **Compact Conversation**: `/compact`

#### `nnn` - Next Task Planning (Analysis & Planning Only)
**Purpose**: Create a comprehensive implementation plan based on gathered context. **NO CODING** - only research, analysis, and planning.

1.  **Check for Recent Context**: If none exists, run `ccc` first.
2.  **Gather All Context**: Analyze the most recent context issue or the specified issue (`nnn #123`).
3.  **Deep Analysis**: Read context, analyze the codebase, research patterns, and identify all affected components.
4.  **Create Comprehensive Plan Issue**: Use a detailed template to outline the problem, research, proposed solution, implementation steps, risks, and success criteria.
5.  **Provide Summary**: Briefly summarize the analysis and the issue number created.

#### `lll` - List Project Status
When you see `lll`, execute relevant `gh` and `git` commands in parallel to get a full overview of the project's state, then provide a visual summary of open issues, recent PRs, and current focus.

#### `rrr` - Retrospective
**Purpose**: Document the session's activities, learnings, and outcomes.

**⚠️ CRITICAL**: The AI Diary and Honest Feedback sections are MANDATORY. These provide essential context and continuous improvement insights. Never skip these sections.

1.  **Gather Session Data**: `git diff --name-only main...HEAD`, `git log --oneline main...HEAD`, and session timestamps.
2.  **Create Retrospective Document**: Use the template to create a markdown file in `retrospectives/` with ALL required sections, especially:
    - **AI Diary**: First-person narrative of the session experience
    - **Honest Feedback**: Frank assessment of what worked and what didn't
3.  **Validate Completeness**: Use the retrospective validation checklist to ensure no sections are skipped.
4.  **Update CLAUDE.md**: Copy any new lessons learned to the main guidelines.
5.  **Link to GitHub**: Commit the retrospective and comment on the relevant issue/PR.

**Time Zone Note**:
-   **PRIMARY TIME ZONE: [Your Time Zone]** - Always show the primary time zone first.
-   UTC time can be included for reference (e.g., in parentheses).
-   Filenames may use UTC for technical consistency.


**Step 3: Create Retrospective Document**
```bash
# Get session date and times
SESSION_DATE=$(date +"%Y-%m-%d")
END_TIME_UTC=$(date -u +"%H:%M")
END_TIME_LOCAL=$(TZ='Asia/Bangkok' date +"%H:%M")

# Create directory structure
mkdir -p retrospectives/$(date +%Y/%m)

# Create retrospective file with auto-filled date/time
cat > retrospectives/$(date +%Y/%m)/${SESSION_DATE}_${END_TIME_UTC//:/-}_retrospective.md << EOF
# Session Retrospective

**Session Date**: ${SESSION_DATE}
**Start Time**: [FILL_START_TIME] GMT+7 ([FILL_START_TIME] UTC)
**End Time**: ${END_TIME_LOCAL} GMT+7 (${END_TIME_UTC} UTC)
**Duration**: ~X minutes
**Primary Focus**: Brief description
**Session Type**: [Feature Development | Bug Fix | Research | Refactoring]
**Current Issue**: #XXX
**Last PR**: #XXX
**Export**: retrospectives/exports/session_${SESSION_DATE}_${END_TIME_UTC//:/-}.md

## Session Summary
[2-3 sentence overview of what was accomplished]

## Timeline
- HH:MM - Started session, reviewed issue #XXX
- HH:MM - [Event]
- HH:MM - [Event]
- HH:MM - Completed implementation

## Technical Details

### Files Modified
```
[paste git diff --name-only output]
```

### Key Code Changes
- Component X: Added Y functionality
- Module Z: Refactored for better performance

### Architecture Decisions
- Decision 1: Rationale
- Decision 2: Rationale

## 📝 AI Diary (REQUIRED - DO NOT SKIP)
**⚠️ MANDATORY: This section provides crucial context for future sessions**
[Write a detailed first-person narrative of your experience during this session. Include:
- Initial understanding and assumptions
- How your approach evolved
- Moments of confusion or clarity
- Decisions made and why
- What surprised you
- Internal thought process]

## What Went Well
- Success 1
- Success 2
- Success 3

## What Could Improve
- Area 1
- Area 2

## Blockers & Resolutions
- **Blocker**: Description
  **Resolution**: How it was solved

## 💭 Honest Feedback (REQUIRED - DO NOT SKIP)
**⚠️ MANDATORY: This section ensures continuous improvement**
[Provide frank, unfiltered assessment of:
- Session effectiveness
- Tool performance and limitations
- Communication clarity
- Process efficiency
- What frustrated you
- What delighted you
- Suggestions for improvement]

## Lessons Learned
- **Pattern**: [Description] - [Why it matters]
- **Mistake**: [What happened] - [How to avoid]
- **Discovery**: [What was learned] - [How to apply]

## Next Steps
- [ ] Immediate task 1
- [ ] Follow-up task 2
- [ ] Future consideration

## Related Resources
- Issue: #XXX
- PR: #XXX
- Export: [session_YYYY-MM-DD_HH-MM.md](../exports/session_YYYY-MM-DD_HH-MM.md)

## ✅ Retrospective Validation Checklist
**BEFORE SAVING, VERIFY ALL REQUIRED SECTIONS ARE COMPLETE:**
- [ ] AI Diary section has detailed narrative (not placeholder)
- [ ] Honest Feedback section has frank assessment (not placeholder)
- [ ] Session Summary is clear and concise
- [ ] Timeline includes actual times and events
- [ ] Technical Details are accurate
- [ ] Lessons Learned has actionable insights
- [ ] Next Steps are specific and achievable

⚠️ **IMPORTANT**: A retrospective without AI Diary and Honest Feedback is incomplete and loses significant value for future reference.
EOF
```

**Step 4: Update CLAUDE.md**
- Copy any new lessons learned to the Lessons Learned section
- Add any new patterns or anti-patterns discovered
- Update user preferences if any were observed

**Step 5: Link to GitHub**
```bash
# Add retrospective to git
git add retrospectives/
git commit -m "docs: Add session retrospective $(date +%Y-%m-%d)"

# Comment on relevant issue/PR with actual path
RETRO_PATH="retrospectives/$(date +%Y/%m)/$(date +%Y-%m-%d_%H-%M)_retrospective.md"
gh issue comment XXX --body "Session retrospective created: ${RETRO_PATH}"
```

**Time Zone Note**: 
- **PRIMARY TIME ZONE: GMT+7 (Bangkok time)** - Always show GMT+7 time first
- UTC time included for reference only (shown in parentheses)
- File names may use UTC for technical consistency
- In all displays and retrospectives, prioritize GMT+7 for user clarity

#### `gogogo` - Execute Planned Implementation
1.  **Find Implementation Issue**: Locate the most recent `plan:` issue.
2.  **Execute Implementation**: Follow the plan step-by-step, making all necessary code changes.
3.  **Test & Verify**: Run all relevant tests and verify the implementation works.
4.  **Commit & Push**: Commit with a descriptive message, push to the feature branch, and create/update the PR.

## Technical Reference

*(This section should be filled out for each specific project)*

### Available Tools

#### Version Control
```bash
# Git operations (safe only)
git status
git add -A
git commit -m "message"
git push origin branch

# GitHub CLI
gh issue create
gh pr create
```

#### Search and Analysis
```bash
# Ripgrep (preferred over grep)
rg "pattern" --type [file-extension]

# Find files
fd "[pattern]"
```

## Development Practices

### Code Standards
-   Follow the established style guide for the language/framework.
-   Enable strict mode and linting where possible.
-   Write clear, self-documenting code and add comments where necessary.
-   Avoid `any` or other weak types in strongly-typed languages.

### Git Commit Format
```
[type]: [brief description]

- What: [specific changes]
- Why: [motivation]
- Impact: [affected areas]

Closes #[issue-number]
```
**Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

### Error Handling Patterns
-   Use `try/catch` blocks for operations that might fail.
-   Provide descriptive error messages.
-   Implement graceful fallbacks in the UI.
-   Use custom error types where appropriate.

## Lessons Learned

*(This section should be continuously updated with project-specific findings)*

### PocketBase-MinIO Integration (2025-08-26)
-   **Scope Management**: Break complex integrations into 1-hour achievable phases - comprehensive doesn't always mean better
-   **Virtual-Hosted-Style DNS**: MinIO requires MINIO_DOMAIN env var and proper /etc/hosts entries for S3 SDK compatibility
-   **Phased Implementation**: Start with storage layer (MinIO), test thoroughly, then add application layer (PocketBase)
-   **Network Architecture**: Separate Docker Compose stacks can communicate via host network ports - simpler than complex Docker networking
-   **Critical Configuration**: Never use IP addresses in S3 endpoints - always use hostnames for virtual-hosted-style requests
-   **Testing Strategy**: Test each component independently before integration - MinIO with MC client first

### Planning & Architecture Patterns (2025-08-26)
-   **Pattern**: Use parallel agents for analyzing different aspects of complex systems
-   **Anti-Pattern**: Creating monolithic plans that try to implement everything at once
-   **Pattern**: Ask "what's the minimum viable first step?" before comprehensive implementation
-   **Pattern**: 1-hour implementation chunks are optimal for maintaining focus and seeing progress

### PocketBase JSVM Quirks (2025-08-24)
-   **Function Scoping**: Functions defined at top level in .pb.js files are NOT accessible in hook handlers - define functions INSIDE the hook handler
-   **VM Isolation**: PocketBase uses separate VMs - "loader" VM for .pb.js files, "executor" VMs for hook execution  
-   **No Timer Functions**: setTimeout, setInterval not available in JSVM - execute code directly
-   **Hook Availability**: onBootstrap is global, but onServe must be accessed via $app.onServe()
-   **API Signatures**: findFirstRecordByFilter doesn't support sorting - use findRecordsByFilter with limit 1
-   **Error Handling**: findFirstRecordByFilter throws "sql: no rows" when no record found - this is expected, wrap in try-catch

### Common Mistakes to Avoid
-   **Creating overly comprehensive initial plans** - Break complex projects into 1-hour phases instead
-   **Using IP addresses in S3 configurations** - Always use hostnames for MinIO to handle virtual-hosted-style requests
-   **Trying to implement everything at once** - Start with minimum viable implementation, test, then expand
-   **Skipping AI Diary and Honest Feedback in retrospectives** - These sections provide crucial context and self-reflection that technical documentation alone cannot capture
-   **Assuming standard Node.js environment in PocketBase JSVM** - It's Goja (Go's JS engine) with limitations
-   **Defining functions at top level expecting them to be accessible in hooks** - They won't be due to VM isolation
-   *Example: Forgetting to update a lockfile after changing dependencies.*
-   *Example: Not checking build logs for warnings that could become errors.*
-   *Example: Making assumptions about API responses instead of checking the spec.*

### Useful Tricks Discovered
-   **Parallel agents for analysis** - Using multiple agents to analyze different aspects speeds up planning significantly
-   **ccc → nnn workflow** - Context capture followed by focused planning creates better structured issues
-   **Phase markers in issues** - Using "Phase 1:", "Phase 2:" helps track incremental progress
-   *Example: Using a specific library feature to simplify complex state.*
-   *Example: A shell command alias that speeds up a common task.*
-   *Example: A design pattern that solved a recurring problem in the codebase.*

### Project-Specific Patterns
-   *Example: The standard way we handle authentication state.*
-   *Example: The required structure for a new API endpoint.*
-   *Example: The component composition pattern used for UI elements.*

### User Preferences (Observed)
-   **Prefers manageable scope** - "i love this - Can be completed in under 1 hour" shows preference for achievable tasks
-   **Values phased approaches** - Recognizes when plans are "too huge" and appreciates splitting work
-   **Appreciates workflow patterns** - Likes using established patterns like "ccc nnn gh flow"
-   *Example: Prefers simple, direct solutions over complex abstractions.*
-   *Example: Values quick iteration and seeing visual progress.*
-   *Example: Appreciates clear, actionable feedback and well-defined tasks.*
-   **Time zone preference: GMT+7 (Bangkok/Asia)**
-   **Repository: laris-co/pocketbase-minio for this project**

## Troubleshooting

### Common Issues

#### Build Failures
```bash
# Check for type errors or syntax issues
[build-command] 2>&1 | grep -A 5 "error"

# Clear cache and reinstall dependencies
rm -rf node_modules .cache dist build
[package-manager] install
```

#### Port Conflicts
```bash
# Find the process using a specific port
lsof -i :[port-number]

# Kill the process
kill -9 [PID]
```

## Appendices

### A. Glossary
*(Add project-specific terms here)*
-   **Term**: Definition.

### B. Quick Command Reference
```bash
# Development
[run-command]          # Start dev server
[test-command]         # Run tests
gh issue create        # Create issue
gh pr create           # Create PR

# Tmux
tmux attach -t dev     # Attach to session
Ctrl+b, d              # Detach from session
```

### C. Environment Checklist
-   [ ] Correct version of [Language/Runtime] installed
-   [ ] [Package Manager] installed
-   [ ] GitHub CLI configured
-   [ ] Tmux installed
-   [ ] Environment variables set
-   [ ] Git configured

**Last Updated**: [Date]
**Version**: 1.0.0
