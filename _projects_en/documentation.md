---
title: Designing a Multi-Channel Documentation Ecosystem
slug: documentation-ecosystem
summary: "A look back at the creation of an intelligent documentation pipeline that transforms a MkDocs knowledge base into a multi-channel publishing system compatible with chatbots and AI: rendered HTML extraction, two-pass synchronisation, automatic link rewriting, intelligent Mermaid diagram handling, and the gradual emergence of a true documentation graph that AI agents can use."
stack:
  [
    MkDocs,
    Material for MkDocs,
    Markdown,
    Python,
    Git,
    GitLab CI/CD,
    API,
    Mermaid,
    YAML,
    HTML,
    Commercial chatbot,
    AI,
  ]
order: 1
featured: true
github_repos:
---

# From a PDF Guide to a Multi-Channel Documentation System

## Overview

The starting point was an approximately 100-page PDF user guide: a static document that was difficult to maintain, filled with ageing screenshots, and poorly suited to a complex product undergoing continuous change.

I chose not to simply rewrite that guide. I designed and built a new documentation ecosystem based on MkDocs, with a single source of truth in Markdown, a modular content architecture, structured metadata, and multiple publishing channels.

The project now contains approximately 118 Markdown files, together with the MkDocs configuration, multimedia assets, technical references, conversion and export scripts, and a project journal. It includes:

- 23 task-oriented guides
- end-to-end business workflows
- concepts and use cases
- troubleshooting guides
- CSV and OpenAPI references
- reusable content blocks
- diagrams, videos, and interactive content
- a publishing pipeline to an external help centre

This is no longer simply a documentation website. It is a system for transforming and distributing knowledge.

---

## The Problem to Solve

The original documentation contained valuable business expertise, but its format prevented that knowledge from being used effectively:

- a single monolithic publication
- duplicated information and gradual divergence between copies
- limited navigation
- updates and validation that were difficult to trace
- screenshots that quickly became obsolete
- insufficient separation between concepts, actions, and references
- limited reuse of shared explanations
- no structure that automated tools could directly use
- no reliable way to publish the same content across several channels

The problem was not merely editorial. It concerned information architecture, maintenance, publishing, and the circulation of knowledge.

---

## A Design Decision

I approached the project as a system-design problem.

The architecture follows a simple chain:

**Markdown and YAML metadata → MkDocs rendering → controlled transformations → website, help centre, and AI tools**

MkDocs is the canonical source. Other formats are not authored separately: they are produced from the same content according to the constraints of each channel.

This decision avoids maintaining several competing versions of the same information and makes it possible to apply principles normally associated with software development to documentation:

- version control with Git
- change reviews
- reproducible builds
- strict validation
- reusable components
- automated deployment
- transformation tracking

---

## An Explicit Content Architecture

The repository is organised according to the actual purpose of each type of content:

- **actions**: complete a specific task
- **concepts**: understand the system’s objects and rules
- **workflows**: follow an end-to-end journey
- **use cases**: connect the product to a business need
- **reference**: consult formats, fields, endpoints, or constraints
- **troubleshooting**: diagnose and resolve a problem
- **\_blocks**: reuse warnings, prerequisites, and cross-cutting explanations

The same goal can be achieved through several modes — UI, CSV, or API. Pages are therefore designed to present a shared objective while clearly separating the steps, errors, and references specific to each method.

Shared blocks also make it possible to correct an explanation once and propagate that correction everywhere it is used.

---

## Metadata as a Structural Layer

Each page can include YAML front matter describing its purpose and relationships:

```yaml
type: action
canonical_id: actions.example.csv
modalities: [csv]
contexts: [implementation]
status: published
chatbot:
  export: true
related:
  - concepts.example
  - workflows.example
```

This metadata does more than classify pages. It makes it possible to:

- select content for publication
- give each page a stable identifier independent of its file path
- connect actions, concepts, workflows, and references
- drive automated transformations
- prepare more precise context retrieval for AI tools
- evolve the documentation without breaking its internal relationships

The content therefore becomes both readable by people and interpretable by machines.

---

## A Pipeline Built on Fully Rendered HTML

The first export prototype appeared straightforward: read the Markdown and send it to an API. That approach did not work correctly.

MkDocs enriches the source during the build process:

- resolving inclusions
- rendering admonitions
- generating tabs
- creating anchors
- applying code highlighting
- transforming links
- integrating Material for MkDocs components

The pipeline therefore extracts the **HTML generated by MkDocs**, not the raw Markdown. The published website becomes an intermediate compilation step whose output can then be adapted to the constraints of the help centre.

This approach ensures that reusable content, Markdown extensions, and visual components have already been resolved before synchronisation.

---

## Two-Pass Synchronisation

Publishing to the help centre relies on a two-pass mechanism.

### Pass One: Create or Update

The pipeline:

1. discovers only content explicitly marked for publication
2. builds the website with MkDocs in strict mode
3. extracts and cleans the rendered HTML
4. creates missing articles
5. updates existing articles
6. stores the mappings between canonical identifiers and remote identifiers

A state file prevents the same articles from being recreated during every run and makes it possible to track their evolution over time.

### Pass Two: Rebuild Relationships

Once all remote identifiers are known, the pipeline processes the articles again to:

- rewrite internal links
- replace MkDocs paths with final URLs
- preserve relationships between articles
- associate related content
- prevent broken links when content is created or moved

This second pass solves a common synchronisation problem: an article cannot point correctly to a remote target until that target has been created.

Intermediate files, resolution tables, rendered HTML, and generated images remain isolated from the source content so that they do not pollute the documentation repository.

---

## The Special Case of Mermaid Diagrams

Mermaid diagrams are essential for representing workflows, decisions, API interactions, and relationships between objects. However, the help centre and its chatbot do not interpret them in the same way as MkDocs.

I therefore designed a dedicated process:

1. automatically detect Mermaid blocks
2. calculate a content hash to identify changes
3. generate the diagram as SVG
4. upload the image to the target platform
5. insert the image into the published article
6. retain a hidden textual representation of the diagram in the HTML

The final step is important. A person sees a properly rendered diagram, while a chatbot or AI agent retains access to the nodes, relationships, labels, and sequences that make up the diagram.

The diagram is therefore no longer merely an illustration. It becomes a documentation object with two representations: visual for people and structured for machines.

---

## The Emergence of a Documentation Graph

The project does not yet rely on a dedicated graph database. Nevertheless, a true documentation graph is already beginning to emerge through the combination of:

- the taxonomy
- each page’s canonical identifier
- `related` metadata
- internal links
- relationships between concepts, actions, and workflows
- the logical structure of Mermaid diagrams

Each page can be considered a node, and each declared link or relationship an edge.

This structure improves navigation for people while also preparing more advanced uses:

- context retrieval for a chatbot
- exploration of related topics
- answers guided by business relationships
- specialised assistants for API integrations
- detection of isolated or insufficiently connected content
- future generation of dynamic documentation journeys

AI compatibility does not come from simply adding a “chatbot” button. It begins with how knowledge is divided, named, and connected.

---

## True Multi-Channel Publishing

From a single source, the system can produce several experiences:

- **MkDocs / GitLab Pages** for rich, structured, navigable documentation
- **conversational chatbot** for querying knowledge in natural language
- **API-focused tools** for finding an endpoint, understanding a model, or interpreting an error
- **video and H5P content** for explanations that benefit from a demonstrative or interactive format

Each channel imposes its own rendering, navigation, and security constraints. The role of the pipeline is to preserve the consistency of the underlying information while adapting its presentation.

---

## CI/CD and Documentation Quality

The repository is integrated with GitLab CI/CD and can be published through GitLab Pages.

The strict build provides a first layer of control: a configuration error, an invalid reference, or incompatible content must be detected before publication. Version control then makes it possible to understand when and why a page changed.

This approach makes the documentation:

- versioned
- testable
- reproducible
- portable
- reviewable
- ready for automated publishing

It also creates the conditions required to welcome other contributors without depending on an ad hoc process understood by only one person.

---

## My Role in the Project

I initiated this change in approach and designed the system from end to end.

My work included:

- deciding to replace the PDF guide with a docs-as-code architecture
- creating a new modular repository
- defining the taxonomy and writing conventions
- designing the metadata and canonical identifiers
- authoring and restructuring the content
- integrating CSV, API, and OpenAPI references
- designing the rendered-HTML export pipeline
- developing the two-pass synchronisation mechanism
- solving internal link rewriting
- designing the multi-format Mermaid processing workflow
- preparing the content for conversational search and AI agents
- organising publication and validation through CI/CD

I used AI assistants as implementation accelerators, particularly to create and evolve the scripts, while remaining responsible for the architecture, business rules, testing, and design decisions.

---

## What the Project Has Already Changed

Without yet claiming a measurable impact on support or product adoption, the structural change is tangible:

- a monolithic PDF has been replaced by approximately 118 modular source files
- knowledge can be corrected and versioned precisely
- shared content can be reused
- UI, CSV, and API journeys can coexist without being confused
- links remain stable across multiple publishing channels
- diagrams can be maintained as code
- the same source can support both human reading and AI retrieval
- the system can evolve without requiring a complete rewrite

Documentation is no longer a deliverable produced after the fact. It becomes a knowledge infrastructure connected to how the product works.

---

## Next Steps

The next stages include:

- progressively extending export to other content families
- formalising onboarding for contributors and administrators
- strengthening automated quality controls
- gaining better visibility into usage and unanswered searches
- making greater use of the documentation graph
- providing AI agents with more precise and explainable context
- bringing documentation, support, training, and developer tools even closer together

The objective remains the same: to produce knowledge that teams can maintain, users can understand, and machines can use.
