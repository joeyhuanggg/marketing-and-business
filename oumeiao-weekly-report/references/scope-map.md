# Session Scope Map

This file defines how the skill should learn the current BI scope at runtime.

## Purpose

The skill should not rely on stale memory about which reports exist under a business scope.
Instead, each reporting task should build a temporary session scope map from the current account's visible BI tree and search results.

## Required Fields

For the selected scope, the map should contain:
- visible scope name
- user-facing business label
- visible child report names
- functional-role matches
- missing expected reports
- confidence level

## Functional Roles

- chain achievement
- channel split
- owner split
- daily trend
- detail drill-down
- optional cross-month support

## Refresh Triggers

Refresh the scope map when:
- a new reporting task starts
- the user logs in again
- the user changes account
- the user says permissions changed
- the visible BI tree differs from earlier in the session

## Confidence Levels

### High

- scope found
- core reports found
- enough support reports found for the task

### Medium

- scope found
- only partial support reports found

### Low

- scope found but important roles are missing
- task is forced to rely on limited evidence
