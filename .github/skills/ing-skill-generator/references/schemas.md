# JSON Schemas for ING Skill Generator

This document defines the JSON schemas used by ing-skill-generator evaluations.

---

## evals.json

Defines the evals for the skill. Located at `workspace/evals/evals.json`.

```json
{
  "skill_name": "ing-skill-generator",
  "evals": [
    {
      "id": 1,
      "name": "baker-repo-full",
      "prompt": "User's example prompt",
      "expected_output": "Description of expected result",
      "files": [],
      "expectations": [
        "Output is a valid SKILL.md file with YAML frontmatter",
        "Contains all 8 required sections"
      ]
    }
  ]
}
```

**Fields:**
- `skill_name`: Name matching the skill's frontmatter
- `evals[].id`: Unique integer identifier
- `evals[].name`: Short identifier for the eval (kebab-case)
- `evals[].prompt`: The task to execute
- `evals[].expected_output`: Human-readable description of success
- `evals[].files`: Optional list of input file paths
- `evals[].expectations`: List of verifiable assertions

---

## grading.json

Output from the grader. Located at `<run-dir>/grading.json`.

```json
{
  "with_skill": {
    "assertions": [
      {
        "text": "Output is a valid SKILL.md file with YAML frontmatter",
        "passed": true,
        "evidence": "File starts with --- and contains name: and description: fields"
      }
    ],
    "pass_rate": 0.83
  },
  "without_skill": {
    "assertions": [
      {
        "text": "Output is a valid SKILL.md file with YAML frontmatter",
        "passed": false,
        "evidence": "File starts with # Baker Framework, no YAML frontmatter present"
      }
    ],
    "pass_rate": 0.17
  },
  "winner": "with_skill",
  "notes": "with_skill output follows ING template structure; without_skill used ad-hoc format"
}
```

---

## ING Skill Output Schema

The generated SKILL.md must follow this structure:

```markdown
---
name: [tool-name-kebab-case]
description: >
  Expert skill for [Tool Name] — an ING-internal framework for [purpose].
  Use this skill when working in any ING Spring Boot / Java 21 project that
  integrates with [Tool Name]. Covers configuration, patterns, and examples.
---

# [Tool Name] — Complete Knowledge Base

## Table of Contents
1. Overview
2. Core Concepts
3. Configuration Reference
4. Code Examples
5. Integration with Other ING Tools
6. Pitfalls & Anti-patterns
7. FAQ
8. Glossary

---

## 1. Overview
[What the tool does, why it exists, current version]

## 2. Core Concepts
[Mental models, architecture, key abstractions]

## 3. Configuration Reference
| Property | Type | Default | Description |
|----------|------|---------|-------------|

## 4. Code Examples
```java
// Verbatim code from source docs
```

## 5. Integration with Other ING Tools
[Connections to Baker, Merak, Kingsroad, etc.]

## 6. Pitfalls & Anti-patterns
❌ **Don't**: [Anti-pattern]
✅ **Do**: [Correct approach]

## 7. FAQ
**Q:** [Question]
**A:** [Answer]

## 8. Glossary
| Term | Definition |
|------|------------|
```

---

## Validation Checklist

When grading generated skills, verify:

1. **Frontmatter**
   - Has `---` delimiters
   - Contains `name:` (kebab-case)
   - Contains `description:` (multi-line with trigger keywords)

2. **Sections**
   - All 8 sections present with correct numbering
   - Table of Contents matches actual sections

3. **Content Quality**
   - Code examples are verbatim (not summarized)
   - Configuration tables have all 4 columns
   - Sparse sections marked with ⚠️
   - Version changes marked with 📌
   - No hyperlinks or external URLs

4. **Version Handling**
   - Latest version identified and used
   - Deprecated content excluded
   - Migration notes preserved if relevant
