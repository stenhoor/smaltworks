# Brand and works: what goes where

**smaltworks decides and records. smalt-brand presents and announces.** Facts flow from smaltworks to smalt-brand;
voice and visual rules flow from smalt-brand to smaltworks.

## smaltworks: direction

- The goal, and the family: which repositories exist, what each is for, where each sits in the architecture.
- Project-wide decisions and tier 1 ADRs; the index of all ADRs and their numbering.
- Standards every repository follows: TypeScript and browsers, npm names and scopes, outside code, plugins (what the
  tiers mean and the compatibility tests behind them), the documentation standard's content rules, licensing,
  repository layout, CI and releases. And the exceptions to them.
- The base context and session rules every repository's `CLAUDE.md` points to.
- Cross-repository work in flight ([WORK.md](../WORK.md)).
- **Releases:** the release plan, the version and compatibility matrix, and the **release notes**.

## smalt-brand: presentation

- The name, the voice, taglines and positioning lines.
- Marks, colours, tokens, type, icons, badges and social cards.
- The site kit (`themes/vitepress/`) and smaltworks.dev.
- **Announcements:** news posts and social posts.

## Release notes and announcements

| | Release notes | Announcement (news) |
| --- | --- | --- |
| **Is** | The record of what shipped | The story of why it matters |
| **For** | People upgrading, and the agents that help them | People deciding whether to try it |
| **Holds** | Every package and version; Breaking first, even when it says "None"; migration steps; compatibility; links to ADRs and PRs | What's new and why, in the family's voice, with visuals; links to the release notes |
| **Lives in** | smaltworks `releases/<version>/release-notes.md`; each product's GitHub release links to it | smalt-brand, published on smaltworks.dev |
| **Rule** | Complete and exact | Never states a fact that isn't in the release notes |

The release notes follow the brand's voice; the announcement follows the release notes' facts.

## Mixed sections

Some sections of smalt-brand's files hold both. Where they do, the direction part moved here and the brand part stays:

| Topic | smaltworks | smalt-brand |
| --- | --- | --- |
| Plugins | Naming, the tiers' meaning, the compatibility tests ([STANDARDS.md](../STANDARDS.md#plugins)) | The tier badges and how they look |
| Documentation | The four sections, reference entries, examples as tests, docs for agents ([STANDARDS.md](../STANDARDS.md#documentation)) | The site kit's components and theme |
| Before 1.0 | The stability promise: breaking changes, exact pins, Breaking first in release notes | The work-in-progress badge and wording |
| smaltty | Its place in the architecture and its rules | Its mark, colour and wordmark |
| How smaltworks works with AI | How sessions work ([context/session-rules.md](../context/session-rules.md)) | The public stance and its wording |

The copies made on 2026-11-15 took whole sections. Trimming the brand detail out of the copied Plugins and
Documentation sections, and the direction detail out of smalt-brand, is part of the removal PR.
