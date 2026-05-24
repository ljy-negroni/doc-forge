# doc-forge

A Claude Code Skill — Full-pipeline toolkit for client-facing software engineering deliverables.

## What is this

doc-forge is designed for developer-client (乙方-甲方) collaboration scenarios. It helps developers guide clients through structured Q&A to clarify requirements, refine business logic, and design databases — then automatically generates all the "edge deliverables": requirement documents, architecture diagrams, promotional images, wireframes, and more.

**Core philosophy:**
- Let clients participate in requirement definition through guided conversation
- Turn ambiguous ideas into structured, professional documents
- Auto-generate diagrams and visuals from confirmed requirements

## Features

### Current

- **Client-facing Structured Q&A** - 6-step guided conversation to help clients clarify: project goals, target users, core features, tech stack, data entities, and business flow
- **Requirement Confirmation** - Auto-detect gaps and risks, prompt clients to supplement or confirm before generating deliverables
- **Document Generation** - Requirement / Design / API documents with template validation and self-check
- **PlantUML Diagrams** - DFD, Use Case, Activity, Sequence diagrams in PNG / SVG / PDF
- **Database Design Assistance** - ER diagrams generated from data entities collected during Q&A
- **Promotional Images** - Claude writes prompts, then calls OpenAI image models (gpt-image-1 / dall-e-3 / dall-e-2)
- **Incremental Updates** - Re-run to update specific modules without regenerating everything

### Planned

- [ ] **Database Schema Design** - Define tables, fields, types, constraints, and relationships interactively; export as SQL DDL
- [ ] **PPT Report Generation** - Auto-generate client presentation decks from confirmed requirements and diagrams
- [ ] **More Diagram Types** - ER diagram, deployment diagram, component diagram
- [ ] **Multi-language Document Output** - Support both Chinese and English document generation

## Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/ljy-negroni/doc-forge.git
cd doc-forge

# 2. Install dependencies
npm install

# 3. Configure environment
cp .env.example .env
# Edit .env and fill in your OPENAI_API_KEY

# 4. Register as Claude Code command
cp skill/SKILL.md ~/.claude/commands/doc-forge.md

# 5. Run in Claude Code
/doc-forge
```

## Typical Workflow

```
1. Developer runs /doc-forge in a meeting with the client
2. Claude asks structured questions → client answers → developer supplements
3. Auto-generate requirement document → client reviews and confirms
4. Auto-generate architecture diagrams (DFD, Use Case, Activity, Sequence)
5. Auto-generate database ER diagram based on discussed data entities
6. Auto-generate promotional images and UI wireframes
7. All deliverables saved with timestamps, ready to send to the client
```

## Project Structure

```
doc-forge/
├── skill/SKILL.md              # Skill main file (Q&A flow + tool invocation)
├── scripts/
│   ├── plantuml-render.js      # PlantUML encoding + rendering (PNG/SVG/PDF)
│   └── image-generate.js       # Multi-model image generation
├── templates/
│   ├── doc-template.md         # Document section structure template
│   └── plantuml-prompts.md     # PlantUML generation prompt templates
├── .env.example                # Environment variable template
└── package.json
```

## Status

Currently under active optimization. Core pipeline (Q&A → document → diagrams → images) is functional.

- [x] Structured Q&A with client-friendly prompts
- [x] PlantUML diagram rendering (PNG / SVG / PDF)
- [x] Multi-model image generation (gpt-image-1 / dall-e-3 / dall-e-2)
- [x] Document compliance self-check
- [x] Incremental update support
- [ ] Database schema design (interactive field definition + SQL export)
- [ ] PPT report generation
- [ ] ER diagram support
- [ ] OpenAI API proxy compatibility
