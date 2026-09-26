# Claude Code Guidelines for Google_4

## Project Overview

**Repository:** ubban/google_4  
**Primary Branch:** main  
**Development Branch:** claude/inspiring-planck-0b4x8s  
**Description:** A lightweight HTML/web project

## Project Structure

```
.
├── README.md          # Project documentation
├── index.html         # Main HTML file
└── CLAUDE.md          # This file - Claude Code guidelines
```

## Development Workflow

### Git Workflow

1. **Primary Development Branch:** `claude/inspiring-planck-0b4x8s`
   - All development work on this repository should be committed to this designated branch
   - Use `git push -u origin claude/inspiring-planck-0b4x8s` for pushing changes
   - If push fails due to network errors, retry with exponential backoff (2s, 4s, 8s, 16s)

2. **Commit Messages**
   - Write clear, descriptive commit messages that explain the "why" not just the "what"
   - Follow this format:
     ```
     Brief summary (under 70 characters)
     
     Detailed explanation if needed (wrapped at 80 chars)
     
     Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
     Claude-Session: <session-url>
     ```
   - Include the attribution lines with author and session information

3. **Pull Requests**
   - DO NOT create pull requests unless explicitly requested
   - When creating a PR, include the attribution footer:
     ```
     🤖 Generated with [Claude Code](https://claude.com/claude-code)
     
     <session-url>
     ```

## Coding Standards

### General Guidelines

- **Clarity over cleverness:** Write code that is easy to understand
- **Minimal comments:** Only add comments when the "why" is non-obvious
- **No premature abstraction:** Don't design for hypothetical future requirements
- **Prefer editing to creating:** Modify existing files rather than creating new ones
- **Security first:** Validate at system boundaries only; trust internal code

### File Organization

- HTML files in the repository root
- Markdown documentation for project information
- Keep the structure simple and flat unless complexity demands organization

## Work Authorization

When working on this repository:

1. **Authorized Actions**
   - Reading and modifying existing files
   - Creating commits to the designated branch
   - Running tests and verification scripts
   - Pushing to `claude/inspiring-planck-0b4x8s` branch

2. **Requires Explicit Permission**
   - Creating pull requests (unless explicitly requested)
   - Force pushing or destructive git operations
   - Modifying CI/CD pipelines
   - Changing repository settings

3. **Always Confirm Before**
   - Deleting files or branches
   - Performing operations that affect shared state
   - Making changes that are hard to reverse

## Claude Code Agent Configuration

### Recommended Tools

- **Glob & Grep:** For code searching and file discovery
- **Edit & Write:** For making code changes
- **Bash:** For running scripts and git commands
- **Agent:** For complex multi-step tasks

### Skill Usage

When applicable, use these skills:
- `/code-review` - For reviewing code changes
- `/run` - For testing changes in the app
- `/loop` - For running recurring tasks

### MCP Servers

- **GitHub:** Available for repository operations
- **Visualize:** Available for creating diagrams and visualizations

## Testing & Validation

Before committing changes:

1. Verify file syntax if applicable
2. Test any interactive features in the HTML
3. Ensure all changes align with project structure
4. Validate that new content doesn't break existing functionality

## Questions & Support

If you have questions about Claude Code usage:
- Run `/help` for general guidance
- Check the [Claude Code documentation](https://claude.ai/code)
- Report issues at https://github.com/anthropics/claude-code/issues

## Session Information

- **Model:** Claude Haiku 4.5
- **Environment:** Remote execution in cloud environment
- **Working Directory:** /home/user/Google_4
- **Scratchpad:** /tmp/claude-0/-home-user-Google-4/[session-id]/scratchpad

---

*This file was generated and is maintained by Claude Code agents to provide consistent guidance for development work on this repository.*
