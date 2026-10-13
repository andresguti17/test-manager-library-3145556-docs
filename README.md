# test-manager-library-3145556-docs

> **Technical test · ADSO cohort 3145556**
> Delivery: **Tuesday, October 13, 2026, at 5:00 p.m.** (17:00, Colombia time).

This repository holds the **spec** for *Manager Library*. It is the only one with content: the other five start empty.

## Instructions

Each apprentice must **fork** the six project repositories and **solve the project
with the proposed spec**.

1. Fork each repository in the table at the end to your GitHub account.
2. Read the spec in [`test-manager-library-3145556-docs`](https://github.com/code-sena/test-manager-library-3145556-docs).
   It ships in three versions: `spec-python/`, `spec-.net/`, and `spec-php/`.
3. Develop in the forks.

## The challenge is built with React and PHP (Laravel)

The spec is written for Python and .NET, but the challenge is **not** done in those languages:

| Layer | Challenge technology |
|---|---|
| Frontend | React |
| Backend | PHP with Laravel |

Whatever the spec defines about the business — stories, acceptance criteria, rules, the
API contract, the data model — is respected. Whatever it defines about technology is translated to React and Laravel.

## The test is not about writing code

The main purpose is to show **performance with SDD** (*Spec-Driven Development*):
how a specification is read, interpreted, and applied to a different stack. Code is the means, not the end.

## Delivery

Whatever is **pushed to the forks before 5:00 p.m.** (17:00) counts. Anything arriving later is not reviewed.
No extensions: push partial progress with small commits during the test.

## What this repository holds

| Folder | Content |
|---|---|
| [`spec-python/`](spec-python/) | The Manager Library spec written for a Python backend |
| [`spec-.net/`](spec-.net/) | The same system specified for a .NET backend |
| [`spec-php/`](spec-php/) | The same system specified for the challenge backend (PHP with Laravel) |
| [`COMMITS.md`](COMMITS.md) | Commit convention and branch workflow for all six repositories |

The three versions describe the **same product** (the same stories, business rules, and
endpoints); the technology decisions change. Documentation is in English in all three tracks
(see Article XI of each `constitution.md`).

## How to read the spec

In any of the three folders, in this order:

1. `constitution.md` — non-negotiable principles.
2. `spec.md` — what the system must do: actors, stories, acceptance criteria, business rules.
3. `plan.md`, `architecture.md`, and `arquitectura-panoramica.md` — how it is built.
4. `data-model.md` and `api-contract.md` — the data model and the API contract.
5. `tasks.md` — what must be done and with what evidence each item counts as done.
6. `adr/` — architecture decisions, with alternatives and consequences.

## The six repositories

| Repository | What goes there |
|---|---|
| [`test-manager-library-3145556-docs`](https://github.com/code-sena/test-manager-library-3145556-docs) | The spec: `spec-python/`, `spec-.net/`, and `spec-php/` |
| [`test-manager-library-3145556-api`](https://github.com/code-sena/test-manager-library-3145556-api) | Backend in PHP (Laravel) |
| [`test-manager-library-3145556-app`](https://github.com/code-sena/test-manager-library-3145556-app) | Frontend in React |
| [`test-manager-library-3145556-page`](https://github.com/code-sena/test-manager-library-3145556-page) | Public static presentation site |
| [`test-manager-library-3145556-infra`](https://github.com/code-sena/test-manager-library-3145556-infra) | Containers, network, volumes, and empty database engine |
| [`test-manager-library-3145556-tool`](https://github.com/code-sena/test-manager-library-3145556-tool) | Utilities: demo-data seeder |
