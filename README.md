# neo-seo-skills

Custom Claude Code skills for SEO analysis, built for real-world workflows with Ahrefs MCP, Tavily, and Firecrawl.

## Skills

| Skill | Description |
|-------|-------------|
| [backlink-analysis](skills/backlink-analysis/) | Deep backlink profile analysis with distributions by authority, website type, link type, mention context, topic, and guest post classification |

## Installation

```bash
# Add individual skill
claude skills add /path/to/neo-seo-skills/skills/backlink-analysis

# Or symlink into your project's .claude/skills/
ln -s /path/to/neo-seo-skills/skills/backlink-analysis .claude/skills/backlink-analysis
```

## Required MCP Servers

- **Ahrefs** — backlink data, referring domains, anchors, organic keywords
- **Firecrawl** — page crawling for link context classification
- **Tavily** — web research (fallback)
