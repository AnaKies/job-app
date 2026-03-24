# Storage Concept

See also: [index.md](./index.md)

## Purpose

This document defines storage responsibilities and storage interpretation rules.

Primary use:

- tell implementation LLMs which data belongs in which storage form
- prevent truth-model confusion between relational, vector, and file storage

## Storage Forms

The approved storage concept uses:

- Supabase Postgres for relational business data
- `pgvector` inside the same Postgres database for semantic vector storage
- object storage for uploaded source documents

Each form serves a different architectural purpose.

Global storage rule:

- relational storage owns operational truth
- vector storage supports semantic retrieval
- object storage owns raw uploaded files

## 1. Supabase Postgres

### Architectural role

Supabase Postgres is the primary relational system of record.

It stores the structured facts of the product:

- what the user uploaded
- which companies and career pages were discovered
- which opportunities exist in normalized form
- what was matched
- what application events happened

### What belongs here

Structured records, metadata, statuses, timestamps, and relationships between core entities belong in relational storage.

### Examples

- `Resume`
  Stores metadata about uploaded resume versions

- `Opportunity`
  Stores normalized job opportunities discovered through scraping

- `Application`
  Stores application lifecycle events such as applied, interview, rejection, or no response

- `Company` and `CareerPage`
  Store discovered companies and their career targets

- `MatchResult`
  Stores match results such as score, reasoning summary, and recommendation state

### Why this matters

Relational storage remains the operational source of truth of the product.
It owns identity, lifecycle, status, and consistency.

Implementation rule:

- if data defines identity, lifecycle, ownership, or business status, store it relationally

## 2. `pgvector`

### Architectural role

`pgvector` extends the same Postgres database with vector-search capability.

It does not replace relational storage.
It augments it with semantic retrieval.

### What belongs here

Embeddings of business-relevant text fragments belong in the vector layer whenever semantic similarity is needed.

### Examples

- `ResumeChunk`
  Semantically meaningful resume sections used for retrieval and matching support

- application-history embeddings
  Semantic representations of past application context used to detect patterns and generate insights

### Why this matters

The vector layer allows the system to retrieve relevant meaning, not only exact relational matches.
This is necessary for:

- resume-to-job relevance support
- history-based insights
- cover-letter support
- retrieval-backed matching context

Implementation rule:

- vector storage must not be treated as canonical record ownership
- embeddings should remain attached to or traceable back to relational business records

## 3. Object Storage

### Architectural role

Object storage holds the original uploaded files that should not live directly inside relational tables.

### What belongs here

Large unstructured source documents belong in object storage.

### Examples

- original resume files such as PDF or Word documents

### Why this matters

The architecture separates source-file storage from relational metadata so that:

- files remain manageable as binary objects
- relational tables stay focused on structured product state
- document metadata and file storage can evolve independently

Implementation rule:

- store raw documents as files
- store document metadata and references relationally

## Combined Relational And Vector Queries

One of the strongest architectural advantages of CeeVee is that relational data and vector data can be combined inside the same database environment.

Because `pgvector` is integrated directly into Supabase Postgres, embeddings can be treated as part of the same storage model rather than as a separate external vector system.

Architecturally, this means:

- relational truth and semantic retrieval remain close to each other
- filtering and ranking can happen in the same database context
- the system avoids cross-system synchronization complexity

Do not infer:

- that vector storage replaces relational queries
- that vector ranking is the same as business truth
- that cross-system syncing is acceptable by default when the architecture already keeps both concerns in one database platform

## Why This Combination Matters

In a split architecture with separate relational and vector databases, the system would need to:

1. query one system for structured records
2. query another system for semantic similarity
3. merge both result sets in application code

That approach increases:

- synchronization complexity
- duplication risk
- stale snapshot risk
- integration overhead

In CeeVee, the architecture keeps both concerns in the same database platform.

Implementation consequence:

- prefer combined relational filtering plus semantic ranking in one database context where possible
- do not split relational and vector truth into separate systems without an explicit architecture change

## Practical Example

The architecture can support queries such as:

“Find opportunities that:

- are still open
- belong to a specific company or subset of companies
- and are most semantically relevant to a resume or resume chunk”

This combines:

- relational filtering
  such as company, status, or freshness

- semantic ranking
  such as meaning-based similarity between job content and resume content

### Why this is important

- Efficiency
  The database can narrow the search space before applying expensive semantic ranking.

- Consistency
  Postgres remains the single source of truth while vector search acts as an augmentation layer.

- Simplicity
  The system avoids synchronizing business truth across separate storage platforms.

## Example Mental Model

A record set such as `ResumeChunk` can be understood architecturally as carrying:

- relational identity and ownership information
- operational metadata
- semantic embedding information

For example, one logical chunk may include:

- which resume it belongs to
- when it was created
- which semantic vector represents its meaning

This does not turn the vector layer into a separate source of truth.
It means the semantic representation lives beside the relational record that it supports.

## Summary Of Responsibilities

| Storage form | Main responsibility | Typical example |
| :--- | :--- | :--- |
| `Supabase Postgres` | Structure, identity, lifecycle, and business relationships | The status of an application such as `interview` |
| `pgvector` | Semantic retrieval and similarity-based ranking | A resume chunk embedding used for matching |
| `Object Storage` | File persistence for original uploaded documents | The original uploaded PDF resume |

## Architectural Rule

The storage concept of CeeVee depends on one strict rule:

- relational storage owns operational product truth
- vector storage augments retrieval and ranking
- object storage owns raw uploaded file persistence

Those responsibilities must remain distinct even when the system combines them in one user-facing workflow.
