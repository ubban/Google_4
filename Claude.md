# Claude Code Configuration for Google_4

## Project Overview

**Google_4** is a web-based project. The repository contains frontend assets including HTML and related resources.

- **Primary directory**: `/home/user/Google_4`
- **Main entry point**: `index.html`
- **Repository**: ubban/google_4

## Development Branch

All development work should be performed on the designated feature branch:
- **Branch**: `claude/inspiring-planck-sfesjt`
- **Base branch**: main (or the repository's default branch)

## Code Style and Conventions

### General Standards
- Write clean, well-organized code following industry best practices
- Default to no comments unless the WHY is non-obvious
- Avoid adding unnecessary abstractions or features beyond what's required
- Prefer explicit, descriptive naming over cryptic abbreviations

### Git Workflow
- Create descriptive commit messages that explain the purpose of changes
- Commits should be focused and atomic (one logical change per commit)
- Include attribution in commit messages (handled by the harness)
- Always push changes to the designated branch using: `git push -u origin claude/inspiring-planck-sfesjt`

### Testing and Validation
- Run manual tests for UI/interactive features before marking complete
- Verify changes don't break existing functionality
- Test the golden path and edge cases for new features
- Avoid relying solely on type checking or test suites to verify feature correctness

## File Management

### Creating and Editing Files
- Prefer editing existing files over creating new ones
- Only create new files when explicitly required
- Place temporary files and intermediate results in the scratchpad directory
- Delete temporary files that are no longer needed

### Security Considerations
- Be cautious of security vulnerabilities (XSS, command injection, SQL injection)
- Validate input at system boundaries only
- Trust internal code and framework guarantees

## Common Patterns

### JavaScript/HTML Development
- Minimize external dependencies
- Load external scripts only from trusted CDNs (cdnjs.cloudflare.com, cdn.jsdelivr.net, unpkg.com)
- Ensure responsive design with proper mobile support

## Permissions and Authorization

- Use `git push -u origin <branch-name>` for pushing code
- Never force-push to shared branches without explicit authorization
- Retry push operations up to 4 times on network failures with exponential backoff (2s, 4s, 8s, 16s)

## Troubleshooting

### Network Issues
If encountering TLS verification errors or proxy issues:
- Check `/root/.ccr/README.md` for proxy configuration
- Run `curl -sS "$HTTPS_PROXY/__agentproxy/status"` for per-tool fixes

### Git Issues
- Use `git status` before destructive operations to avoid losing work
- Prefer merging over rebasing on shared branches
- Keep commits focused on a single logical change

## Contact and Support

For questions about Claude Code functionality:
- Use `/help` for general assistance
- Report issues at https://github.com/anthropics/claude-code/issues
- Consult repository-specific documentation and skill files

---

**Last updated**: 2026-09-28  
**Maintained by**: Claude Code Agent
