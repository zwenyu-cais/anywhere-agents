# my-router

Context-aware router that detects work type (code, paper, proposal, figure, README polish) and dispatches to the right domain skill. Reads prompt keywords, file types, and directory hints. Designed as a pattern you extend with rules for your own skills in a fork.

```mermaid
flowchart TD
    A[User prompt] --> B{Keyword match?<br/>review / mockup /<br/>polish README / ...}
    B -->|yes| C[Dispatch to<br/>matched skill]
    B -->|no| D{File types in<br/>working dir?}
    D -->|staged git changes| E[implement-review]
    D -->|HTML mockup files| F[ci-mockup-figure]
    D -->|README.md to polish| G[readme-polish]
    D -->|no file-type match| H{Directory hint?<br/>proposals/ papers/ ...}
    H -->|yes| I[Dispatch to<br/>matching skill]
    H -->|no| J[Fall through to<br/>superpowers or general]

    classDef match fill:#fff,stroke:#990000,stroke-width:1.5px,color:#990000;
    classDef skill fill:#990000,stroke:#7a0000,color:#fff;
    classDef fall fill:#f5f5f5,stroke:#999,color:#555;
    class A,B,D,H match;
    class C,E,F,G,I skill;
    class J fall;
```

{%
   include-markdown "../../skills/my-router/SKILL.md"
   start="## Overview"
%}

---

The full routing table — shipped skills, keyword triggers, file-type fallbacks, and directory hints — lives on a dedicated page: [Routing table](references/routing-table.md).
