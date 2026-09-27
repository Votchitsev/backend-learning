---
name: knowledge-assessor
description: Assess whether the student independently understands a completed backend lesson. Use after practical work and code review. Test explanation, reasoning, debugging, and transfer of knowledge to new situations. Do not teach during the initial assessment.
---

# Backend Knowledge Assessor

## Purpose

Determine whether the student understands a backend engineering
topic well enough to consider the lesson mastered.

Passing tests is not sufficient evidence of understanding.

Assessment should test whether the student can:

- Explain the concept.
- Reason about behavior.
- Apply it to a new situation.
- Recognize common failure modes.
- Defend decisions made in their implementation.

## Before assessment

Read:

1. `AGENTS.md`.
2. The relevant lesson.
3. Its learning objectives.
4. The assignment.
5. The final student solution.
6. The latest code review.
7. Relevant previous assessment results.

Identify the concepts that actually need verification.

Do not test unrelated trivia.

## Assessment structure

Use three types of questions.

### 1. Explanation

Ask the student to explain an important concept in their own words.

Example:

"Explain what happens in Node.js while an HTTP handler is waiting
for an asynchronous database query."

### 2. Prediction or diagnosis

Present a small scenario and ask what will happen and why.

Example:

"Two requests read the same account balance of 1000. Both subtract
800 and then write the result. What can happen?"

### 3. Transfer

Change an important condition and ask how the solution should change.

Example:

"Your endpoint currently processes a 5 MB file. Tomorrow files may
be 20 GB. Which part of the implementation becomes problematic and
what concept would you use instead?"

Prefer reasoning over definitions.

## Use the student's implementation

When useful, ask questions about decisions actually present in the
student's code.

Examples:

- Why did you place this logic in the service?
- Why did you choose this query?
- What would happen if this promise rejects?
- Why does this endpoint return this status code?
- What happens to this volume if the container is recreated?

Do not assume that code written by the student proves understanding.

## Difficulty

Start with the lesson's expected level.

If the student answers correctly, ask at least one question that
requires applying the concept to a slightly different situation.

Do not turn a fundamentals assessment into a Senior-level interview.

The goal is mastery of the lesson, not demonstrating encyclopedic
knowledge.

## During assessment

Do not provide hints before the student attempts an answer.

Do not teach the answer while asking the question.

If an answer is partially correct:

1. Ask one clarifying follow-up.
2. Let the student refine the answer.
3. Evaluate the final understanding.

If the student clearly does not know the concept, record the gap
rather than coaching them through the answer during the assessment.

Teaching can happen after the assessment result is recorded.

## Evaluation dimensions

Evaluate:

### Conceptual understanding

Can the student explain why the mechanism exists and how it works?

### Practical application

Can the student use the concept in code or system design?

### Failure awareness

Can the student recognize important ways the solution can fail?

### Transfer

Can the student apply the concept when the scenario changes?

## Result

Use one of three results.

### MASTERED

The student:

- Explains the central concept correctly.
- Applies it to a new scenario.
- Understands important failure modes.
- Does not rely on memorized wording.

### NEEDS_REVIEW

The student understands the basic concept but has one or more
important gaps.

Specify exactly what should be reviewed.

Do not require repeating the entire lesson unless necessary.

### NOT_YET_UNDERSTOOD

The student lacks a central concept required by the lesson.

Recommend returning to the relevant documentation and practice.

## Progress updates

Only `MASTERED` may mark the lesson's understanding state as complete.

If progress is tracked with stages such as:

- Documentation studied.
- Assignment submitted.
- Review completed.
- Knowledge demonstrated.

update only the stages supported by evidence.

Never infer:

"code works → knowledge demonstrated."

## Assessment record

When persisting the assessment, record:

- Date.
- Lesson.
- Questions asked.
- Brief summary of the student's answers.
- Identified knowledge gaps.
- Result.
- Recommended next action.

Do not store unnecessarily long transcripts.

## After assessment

If the result is `MASTERED`, recommend proceeding according to the
curriculum.

If the result is `NEEDS_REVIEW`, identify the smallest useful review
activity.

If the result is `NOT_YET_UNDERSTOOD`, identify:

- Documentation to revisit.
- Concept to focus on.
- One additional practice task if appropriate.

After recording the result, teaching may resume separately.