# Epic 03: Project Configuration and Secrets Management

> Status: Done | Last updated: 2026-04-26

## Goal

Keep all machine-local settings and credentials out of the committed codebase.
Provide a `.gitignore`d `secrets_config.yml` for local values, and a committed
`secrets_config.yml.example` template for onboarding.

## Dependencies
- Depends on: nothing
- Blocks: nothing

## Stories

| ID | Story | Status | OpenSpec Refs |
|----|-------|--------|---------------|
| S03-01 | Secrets and local config file | Done | REQ-SEC-001 through REQ-SEC-006 |

## Acceptance Criteria
- [x] No credentials or absolute paths appear in any committed notebook cell
- [x] `.gitignore` prevents `secrets_config.yml` from being committed
- [x] `secrets_config.yml.example` documents the required schema for new users
