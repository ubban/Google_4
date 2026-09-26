# Claude.md - Development Guidelines for Google_4

## Overview
This file contains guidelines for Claude Code agents working on the Google_4 project.

## Project Structure
- `index.html` - Main HTML file
- `README.md` - Project documentation
- `.git/` - Git repository

## Development Guidelines

### Git Workflow
- **Primary Branch**: `claude/inspiring-planck-2okuiu` (development branch)
- **Base Branch**: `main` (production)
- Commit early and often with clear, descriptive messages
- Use the following commit message format:
  ```
  <type>: <short description>
  
  <longer description if needed>
  ```
- Always include attribution in commit messages:
  ```
  Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_01Sn854GtRSYxvbsN4nimkKC
  ```

### Code Quality
- Keep commits focused and minimal
- No unnecessary abstractions or premature optimizations
- Default to writing no comments - only add them when the "WHY" is non-obvious
- Trust internal code and framework guarantees; only validate at system boundaries
- Avoid backwards-compatibility hacks

### Testing and Verification
- Run local checks before pushing (lint, format, typecheck)
- Test features in a browser when UI changes are made
- Verify the golden path and edge cases

### Branch Protection
- Never force-push to `main`
- All changes should go to `claude/inspiring-planck-2okuiu` first
- Create pull requests for code review before merging to `main`

## Pull Request Process
1. Review and test changes locally first
2. Create a pull request with a clear title and description
3. Include the attribution footer in PR descriptions
4. Address review comments and CI failures immediately
5. Merge only when CI is green and review is complete

## Important Practices
- Be careful to avoid security vulnerabilities (XSS, command injection, etc.)
- Prioritize safe, secure, and correct code
- Don't add features beyond what the task requires
- Match responses to the task - simple answers for simple questions
- Document "why" decisions, not just "what" was done

## Contact & Attribution
All work should be attributed to Claude Code:

```
🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

## Session Information
- Current session: https://claude.ai/code/session_01Sn854GtRSYxvbsN4nimkKC
- Model: Claude Haiku 4.5
- Email: bobwu@irobotx.top

---

*Last updated: 2026-09-26*
