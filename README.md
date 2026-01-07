# farmakan's Skills Collection

A collection of Claude Code plugins 

## branch-review

Pre-PR code review plugin for Claude Code. Reviews all changes on your branch compared to main before creating a pull request.

```bash
/plugin marketplace add farmakan/claude-plugins
/plugin install branch-review@farmakan-plugins
```

This is a **command** - invoke with `/branch-review:branch-review`

Reviews for: bugs, security vulnerabilities (OWASP top 10), CLAUDE.md compliance, Filters false positives and groups by severity.

## License

MIT
