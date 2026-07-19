---
title: Traceability Explorer
slug: traceability-explorer
summary: "A Python tool that starts from a purchase order, recursively reconstructs a traceability chain distributed across multiple API environments, generates an explainable Mermaid graph, and flags data inconsistencies before validation."
stack: [Python, REST API, JSON, Mermaid, Graph Traversal, Data Validation, OOP]
order: 0
featured: true
github_repos:
---

# Traceability Explorer: Reconstructing and Validating a Traceability Chain

## Overview

Traceability Explorer is a Python tool designed to automate the analysis of traceability tests performed on a SaaS platform.

The starting point is a purchase order. From that transaction, the tool explores several API environments, retrieves the associated events, and then recursively follows every reference to lots, items, sites, and other transactions.

The objective is not merely to collect objects. It is to reconstruct a coherent story:

- what was ordered?
- what was received?
- which lots were transformed?
- which lots were produced or shipped?
- do the items match between the order and its fulfilment?
- can lot continuity be demonstrated?
- do the sites and transactions follow the expected structure?

The result takes the form of a Mermaid graph accompanied by explicit checks. The tool is intended to detect problems before human validation and reduce an investigative task that can take approximately four hours per test.

---

## The Initial Problem

Validating a supplier test was essentially a manual process.

It involved navigating between several environments, searching for an order, finding the events that might be linked to it, opening each object, recording its identifiers, and then repeating the process with every newly discovered object.

This method had several limitations:

- slow and repetitive exploration
- risk of overlooking an indirectly referenced object
- difficulty distinguishing a genuine transformation from simple lot continuity
- incomplete representation of many-to-many relationships
- late detection of inconsistencies
- dependence on informal knowledge of the data model
- results that were difficult to share or reproduce

A chain can appear correct when its objects are examined separately while remaining inconsistent as a whole.

---

## A Chain Distributed Across Two Environments

The exploration begins in the ordering organisation’s environment, referred to here as **T0**. This is where the purchase order used as the entry point is located.

Execution events are then retrieved from the supplier’s environment, referred to as **T1**.

The program therefore uses two separate API connections:

1. **T0 connection**: retrieve the order, its items, and the relevant sites
2. **T1 connection**: explore receiving, transformation, and shipping events together with their related objects

This separation matters. A single business process crosses several authorisation spaces, and each piece of information must be retrieved within the context in which it is actually accessible.

Connection settings are externalised in a `.env` file so that sensitive credentials are never placed in the source code.

---

## The Domain Model

The tool translates the traceability model into several families of objects.

### Transactions

Transactions provide the administrative context:

- purchase orders
- work orders
- other commercial or logistical references

### Events

Events represent what physically happens to the goods:

- **Receiving**: receipt of a lot
- **Transforming**: consumption of input lots and creation of output lots
- **Shipping**: shipment of a lot

### Reference Data

The chain also relies on:

- items
- lots
- sites

An event can reference several transactions, and a transaction can appear in several events. The relationship is therefore not a simple tree: it is many-to-many.

Transformations add another dimension. They do not merely connect an event to a lot; they describe the transition from one or more input lots to one or more output lots.

---

## Why the Exploration Must Be Recursive

An exploration limited to the events directly linked to the order is not sufficient.

Each discovered object can contain new references:

- an order references items and sites
- an event references transactions and traceable objects
- a lot references an item
- a transformation connects input and output lots
- a transaction can lead to other events

Traceability Explorer therefore maintains a queue of objects to inspect. Each new reference is added to the queue, then retrieved and analysed in turn. Exploration continues until no new object can be discovered.

The process resembles a breadth-first graph traversal:

1. retrieve the entry purchase order
2. extract its initial references
3. find candidate events in the supplier environment
4. filter locally when remote search capabilities are insufficient
5. add every referenced object to the exploration queue
6. retrieve each object that has not yet been seen
7. repeat until the queue is exhausted

In-memory indexes prevent unnecessary API calls and avoid loops when the same object is encountered more than once.

---

## Dynamically Resolving Traceable Objects

A traceable-object reference does not always reveal immediately whether it identifies a lot or an item.

The resolution engine therefore attempts to determine the object’s actual type and records it in the appropriate model.

This distinction is essential:

- an **item** describes what a product is
- a **lot** represents a traceable occurrence of that product

Using items directly in events can make a chain appear complete while removing the physical continuity required for genuine traceability.

The tool therefore explicitly flags cases in which items are used as traceable objects instead of lots.

---

## A Lot Is Not a Single Node

One of the project’s main challenges lies in representing lots.

The same lot can appear several times in the chain:

- as a received lot
- as an input to a transformation
- as an output from a transformation
- as a shipped lot
- across several successive events

Merging all these appearances into a single Mermaid node would remove the order of operations and create misleading shortcuts.

Traceability Explorer therefore creates a **separate occurrence for each role played by a lot within an event**.

Each occurrence has:

- a unique graphical identifier
- the lot to which it corresponds
- the event in which it appears
- its role
- its position in time

The graph can therefore represent both the lot’s stable identity and its different uses over time.

---

## Distinguishing Transformation, Continuity, and Reference

The graph uses several types of links.

### Transformation

A solid arrow represents a genuine transformation:

```text
input lot --> output lot
```

The material or product changes state during a transformation event.

### Continuity

A continuity link connects two occurrences of the same lot:

```text
received lot --- shipped lot
```

The lot remains the same, but appears at two different moments or in two different roles.

Occurrences considered to be sources include:

- a received lot
- a lot produced by a transformation

Occurrences considered to be uses include:

- a shipped lot
- a lot consumed by a transformation

For each use, the algorithm searches for the most plausible earlier source. This rule preserves chronological continuity without confusing movement, storage, and transformation.

### Reference

A third link style represents documentary or descriptive relationships:

- event to transaction
- lot to item
- transaction to order

These links explain the context without being interpreted as physical movement.

---

## The Checks Produced

The first set of validations focuses on simple but structural rules.

### Site Structure

The tool checks whether the sites associated with the order and events follow the expected pattern between issuing, origin, and destination sites.

It does not automatically make a business decision: it presents discrepancies for review.

### Items or Lots

A warning is generated when an event uses an item rather than a lot as its traceable object.

### Matching the Order and Its Fulfilment

Ordered items are compared with the items found in received, transformed, or shipped lots.

The objective is to identify:

- an ordered item missing from the fulfilment data
- an item present in the fulfilment data with no clear match in the order
- confusion between an item identifier and a lot identifier

### Lot Continuity

Every use of a lot should be traceable to a credible earlier source.

A missing source, inconsistent chronology, or isolated fragment becomes immediately visible in the graph and validation report.

---

## An Explainable Mermaid Visualisation

The Mermaid output is organised into separate subgraphs:

- orders
- receiving events
- transformation events
- shipping events

This structure makes it possible to read the diagram as a sequence of operations rather than as a simple map of objects.

Nodes display only the information required for the analysis, while link styles distinguish:

- transformations
- continuity of the same lot
- administrative references

Mermaid was selected for several reasons:

- producing a textual, versionable result
- enabling rapid inspection
- making a test case easy to share
- preserving a reproducible representation
- exposing anomalies without requiring a complex graphical interface

An optional JSON diagnostic output can complement the `.mmd` file, making it possible to inspect the collected objects and the decisions made by the algorithm.

---

## Program Architecture

The project is organised into several layers.

### Connections and Roles

`ApiConnection` and `TenantRole` describe the available environments and prevent the responsibilities of T0 and T1 from being mixed.

### Domain Model

Classes represent the main objects:

- purchase orders and work orders
- transactions
- receiving, transformation, and shipping events
- lots, items, and sites

Enumerations describe, among other things:

- event types
- traceable-entity types
- lot-occurrence roles
- Mermaid link types

### Exploration Context

`ExplorationContext` centralises:

- connections
- objects already retrieved
- the work queue
- lot occurrences
- continuity links
- warnings and validation results

### Services

Separate services handle:

- recursive exploration
- reference resolution
- occurrence reconstruction
- consistency checks
- Mermaid generation
- diagnostic export

This separation makes the program easier to test and allows business rules to be adapted without rewriting the exploration engine.

---

## Current Status

An initial executable V1 has been assembled.

It is structured to:

- load two connections from the configuration
- retrieve a purchase order as the starting point
- recursively explore related objects
- build a unified in-memory context
- produce a Mermaid diagram
- generate JSON diagnostic output
- flag initial inconsistencies involving sites, items, and lots

The project remains a technical prototype. Adapters for the exact schemas of different API versions, automated tests, and the final report presentation still need to be consolidated.

It does not correct any data and does not replace the judgement of the person responsible for validation. Its role is to make the investigation faster, more complete, and more explainable.

The projected saving of approximately four hours per test is therefore an operational target to be confirmed, not an established measurement.

---

## What This Project Demonstrates

Traceability Explorer brings together several aspects of my work:

- translating a complex business model into software objects
- recursively exploring a graph distributed across several APIs
- working with many-to-many relationships
- managing the identity and temporal occurrences of the same entity
- designing explainable checks rather than a black box
- producing a visualisation that supports decision-making
- automating an operational task born from a real need

The project also illustrates how I work when real-world data does not match a perfectly clean theoretical model: I preserve ambiguities, make them visible, and provide the person validating the data with the evidence needed to make a decision.

---

## Next Steps

Planned developments include:

- stabilising the API adapters
- adding unit tests and synthetic datasets
- enriching chronological checks
- detecting cycles and disconnected fragments more systematically
- producing a readable HTML report alongside Mermaid
- distinguishing blocking errors from warnings
- comparing several runs of the same test
- measuring the time actually saved
- making it easier to add new rules without modifying the core engine

The ultimate objective is to turn an ad hoc investigation into a reproducible, traceable, and assisted analysis — while leaving the final decision to a human being.
