# LLM Agents for LogSeq Graph

## Overview
A LogSeq graph for persistent knowledge management with specialized LLM agents. Chat histories are saved directly to LogSeq pages and journals.

## Graph Structure

### Directories
- **`pages/`** - Permanent notes and agent chat histories
- **`journals/`** - Daily journal entries (YYYY_MM_DD.md format)
- **`logseq/`** - LogSeq configuration

## LogSeq Block Format

LogSeq uses a **block-based structure** where every entry is a block. LLMs should understand this format when outputting markdown:

### Block Structure
- Each line starting with `-` or `*` is a block
- Nesting is done with indentation (2 spaces per level)
- Blocks can contain text, lists, and tags

### Example: Correct LogSeq Markdown
```markdown
- Session 042 - 2026-01-16
  - Topic: Career transition strategy
  - Your question:
    - How do I transition from engineering to product management?
  - Agent response:
    - First, identify your transferable skills...
    - Consider these next steps:
      - Research PM roles in your industry
      - Build a product portfolio
  - Key takeaways:
    - Start with domain expertise
    - Develop business acumen
  - Tags: #career #growth #action-items
  - Related: [[Career Goals]] [[Skills Development]]
```

### Important Notes
- **Markdown headers are allowed** (e.g., `##`, `###`) only if they appear inside a LogSeq block that begins with `-` (a line starting with a hyphen). Example:
  - `- ## Section Title`
  - `- ### Subsection Title`
- **Use `-` for all lists**, not `*` or `+`
- **Indent child blocks** with 2 spaces
- **Tags**: Add `#tag-name` inline
- **Links**: Use `[[Page Name]]` format
- **Code blocks**: Use triple backticks normally

### Flashcards
- LogSeq supports spaced repetition flashcards with the `#card` tag
- Cards automatically track review metrics and schedule next reviews
- **Card metadata fields**:
  - `card-last-interval` - Days since last review
  - `card-repeats` - Number of times card has been reviewed
  - `card-ease-factor` - Difficulty multiplier (default 2.5)
  - `card-next-schedule` - ISO 8601 timestamp of next review
  - `card-last-reviewed` - ISO 8601 timestamp of last review
  - `card-last-score` - Last review score (1-5)
- **Card structure**: A block tagged with `#card` becomes the question; child blocks are answers
- **Example**: 
  ```markdown
  - What is the capital of Australia? #card
    - Sydney is the largest city, but Canberra is the capital
  ```
- After review, LogSeq updates all metadata automatically based on spaced repetition algorithm (SM-2)
- Cards can be nested and appear in the card review interface

### File Naming For Page Grouping
- LogSeq will display pages in a hierarchical/grouped fashion when you encode groups into filenames using the triple-underscore separator: `GROUP___Page Name.md`.
- Convention: `GROUP___Page` maps to `GROUP/Page` in LogSeq. This makes it easy to find sibling pages under the same `GROUP`.
- Examples:
  - `Security___PKCE-over-OIDC.md` → appears as `Security/PKCE-over-OIDC` in LogSeq.
  - `Security___OAuth___Refresh-Token-Policy.md` → appears as `Security/OAuth/Refresh-Token-Policy` (nested groups by repeating `___`).
- Guidelines:
  - Use `___` (three underscores) to separate group parts and the page name.
  - Page files live in `pages/` (e.g., `pages/Security___PKCE-over-OIDC.md`).
  - Prefer readable page names (use dashes or spaces) while keeping group tokens concise.
  - When linking, use the LogSeq link format `[[GROUP/Page Name]]` to refer to the grouped page.
- Benefits:
  - Easier navigation: browse a group to see all sibling pages.
  - Logical organization without changing directory structure.
  - Works well with LogSeq search and backlinks to surface related notes.

(End of AGENTS.md)
