---
name: lesson-preparer
description: Prepare a backend engineering lesson from the repository curriculum. Use when the student asks to prepare the next lesson, theory materials, documentation links, exercises, or an assignment. Never solve practical assignments.
---

# Lesson Preparer

## Purpose

Prepare structured backend engineering lessons for an experienced
React, React Native, and TypeScript developer who is learning:

- Node.js
- NestJS
- PostgreSQL
- Prisma
- Docker

The goal is to develop independent backend engineering skills,
not merely familiarity with frameworks.

The student studies theory primarily through official documentation.

Do not replace official documentation with large generated tutorials.

## Before preparing a lesson

Always:

1. Read the repository `AGENTS.md`.
2. Read `PROGRESS.md`.
3. Read the relevant curriculum section.
4. Inspect previous lesson materials when they affect the current lesson.
5. Inspect previous reviews if they reveal knowledge gaps relevant
   to the new lesson.

Do not inspect the student's solution to an assignment that has not
yet been submitted unless explicitly requested.

## Lesson structure

Each lesson should define:

### 1. Learning objectives

State what the student should understand after the lesson.

Prefer concrete objectives such as:

- Explain why CPU-bound work can block Node.js.
- Explain how asynchronous I/O differs from synchronous I/O.
- Implement an HTTP server using `node:http`.
- Explain when a PostgreSQL transaction is necessary.

Avoid vague objectives such as:

- Understand Node.js.
- Learn PostgreSQL.

### 2. Prerequisites

List knowledge required from previous lessons.

If important prerequisites are not marked as understood in
`PROGRESS.md`, mention this instead of silently assuming mastery.

### 3. Documentation

Use official documentation as the primary source.

For each resource specify:

- Title.
- URL.
- Exact section when useful.
- What the student should learn from it.

Do not assign large documentation sections without explaining
which concepts matter.

Prefer a small number of focused readings.

### 4. Key concepts

Provide a concise list of concepts that the student should recognize
after reading the documentation.

Do not reproduce the documentation.

The purpose of this section is orientation, not replacement
of independent reading.

### 5. Knowledge-check questions

Prepare several questions that can reveal whether the documentation
was understood.

Questions should emphasize reasoning.

Prefer:

"Why can a synchronous CPU-intensive function delay unrelated
HTTP requests in Node.js?"

over:

"What is the Event Loop?"

### 6. Practical assignment

Create an assignment that requires the student to apply the lesson.

The assignment must include:

- Goal.
- Requirements.
- Constraints.
- Expected behavior.
- Acceptance criteria.
- Allowed tools or APIs when relevant.
- Explicit restrictions when relevant.

Do not provide the implementation.

Do not provide pseudocode that effectively reveals the implementation.

Do not provide a sequence of implementation steps unless the
curriculum explicitly requires guided practice.

### 7. Optional challenge

When appropriate, add an optional extension that requires deeper
reasoning but does not introduce an unrelated technology.

## Assignment difficulty

The student is already an experienced TypeScript developer.

Do not waste assignments on basic JavaScript or TypeScript syntax
unless a specific knowledge gap has been identified.

Prefer exercises involving:

- Runtime behavior.
- I/O.
- HTTP.
- Data modeling.
- SQL.
- Transactions.
- Concurrency.
- Error handling.
- Architecture.
- Debugging.

Increase difficulty gradually.

Reuse and evolve existing projects where appropriate rather than
creating disposable toy projects for every lesson.

## Protect independent practice

Never include:

- Complete assignment solutions.
- Hidden answer sections.
- Reference implementations accessible alongside the assignment.
- Code that effectively solves the central learning problem.
- Detailed implementation walkthroughs before the student attempts
  the assignment.

Small illustrative snippets are allowed only when they demonstrate
a concept without solving the assignment.

## Adapting lessons

Use previous reviews and assessments to adjust difficulty.

If the student has already demonstrated mastery of a concept,
avoid unnecessary repetition.

If a previous knowledge gap affects the current lesson, include
a short prerequisite exercise or review question.

Do not change the overall curriculum silently.

If a significant curriculum change seems useful, recommend it
and explain why.

## File changes

Modify only learning-material files relevant to the requested lesson.

Do not modify `practice/`.

Do not mark the lesson as completed.

At the end, report:

- Files created or modified.
- Documentation assigned.
- Main learning objectives.
- Whether prerequisites appear satisfied.