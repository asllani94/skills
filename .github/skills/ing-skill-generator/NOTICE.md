# Attribution Notice

This skill (`ing-skill-generator`) is a derivative work based on `skill-creator`,
which is licensed under the Apache License, Version 2.0.

## Original Work

- **Name:** skill-creator
- **Source:** GitHub Copilot CLI Skills
- **License:** Apache License 2.0

## Modifications

The following modifications were made to create `ing-skill-generator`:

1. **SKILL.md** — Substantially rewritten to focus on generating skills from
   ING-internal documentation repositories. Added:
   - ING-specific 8-section template (Overview through Glossary)
   - Naming, versioning, and content extraction rules for ING frameworks
   - Local repository analysis workflow (vs. general skill creation)
   - Target audience guidance for ING senior engineers
   - References to ING frameworks (Baker, Merak, Kingsroad)

2. **references/schemas.md** — New file documenting ING skill output schema
   and validation checklist.

3. **Scripts and agents** — Copied from skill-creator without modification:
   - `agents/grader.md`
   - `agents/comparator.md`
   - `agents/analyzer.md`
   - `scripts/*.py`
   - `eval-viewer/*`
   - `assets/eval_review.html`

## License

This derivative work is also licensed under the Apache License, Version 2.0.
See LICENSE.txt for the full license text.

---

Copyright 2026 Arnold Asllani

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
