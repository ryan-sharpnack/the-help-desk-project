# IT Support Ticket Library

This directory contains practical IT Support ticket writeups developed through hands-on troubleshooting, homelab exercises, researched scenarios, and community contributions.

Each ticket documents the investigation and resolution process, including the reported problem, business impact, diagnostic methodology, root cause, corrective actions, and verification.

The objective is to develop practical troubleshooting capabilities while creating a resource that aspiring IT professionals, experienced practitioners, and hiring managers can learn from.

## Ticket Library Structure

Tickets are organized into three competency levels:

| Level | Tickets | Focus |
|---|---:|---|
| [L1 — Foundational IT Support](L1/) | 12 | Common end-user computing and entry-level troubleshooting |
| [L2 — Intermediate IT Support](L2/) | 12 | Enterprise systems, identity, infrastructure services, and networking |
| [L3 — Advanced IT Support](L3/) | 6 | Complex infrastructure incidents, root cause analysis, and service restoration |

Each level contains its own README with a ticket index, learning objectives, and completion status.

## What Belongs Here

- Practical IT Support incidents and troubleshooting exercises.
- Issues involving Windows, Linux, networking, hardware, identity and access, and infrastructure services.
- Writeups explaining the diagnostic process and why particular troubleshooting steps were taken.
- Verified resolutions and supporting technical evidence.
- Community-contributed tickets based on authorized experience.

Scenarios that have not yet been resolved or are intended for others to investigate belong in the [Scenarios](../scenarios/) directory instead.

## Ticket Writeup Requirements

Use the standardized [Ticket Writeup Template](TEMPLATE.md) when documenting a ticket.

Each writeup should include:

- Incident description and business impact.
- Environment and initial observations.
- Symptoms and troubleshooting methodology.
- Diagnostic commands and supporting evidence.
- Root cause and resolution.
- Verification and testing.
- User communication and closure, where applicable.
- Lessons learned and escalation considerations.

Clearly identify the environment and nature of the experience:

- **Homelab:** The incident was reproduced and investigated in a personal lab environment.
- **Production (Authorized):** The work was performed in an authorized production environment and can be shared without disclosing confidential information.
- **Research (Documented):** The scenario was researched and documented but not independently reproduced.

Do not present researched scenarios as hands-on experience.

## Naming Convention

Use the following naming convention for level-based tickets:

`L1-001-short-description.md`

Examples:

- `L1-001-windows-user-cannot-sign-in.md`
- `L2-013-domain-authentication-failure.md`
- `L3-025-internal-dns-failure.md`

Use the established ticket ID and title when documenting each exercise.

## How to Contribute

1. Review the existing ticket library to avoid unnecessary duplication.
2. Select an appropriate ticket or propose a new scenario.
3. Copy the [Ticket Writeup Template](TEMPLATE.md).
4. Complete the writeup and include relevant supporting evidence.
5. Open a Pull Request following the instructions in [CONTRIBUTING.md](../CONTRIBUTING.md).

Community contributions should reflect genuine technical experience, authorized research, or clearly identified documented scenarios.

## Before You Submit

- Remove passwords, API keys, tokens, and other secrets.
- Redact or replace sensitive IP addresses, hostnames, usernames, and personal or employer information.
- Verify commands and configurations where possible.
- Ensure that you have permission to share all submitted material.
- Accurately represent the environment and nature of the work.
- Credit external documentation and resources.

By submitting, you agree to the contribution guidelines in [CONTRIBUTING.md](../CONTRIBUTING.md).

## Difficulty Levels

| Level | Meaning |
|---|---|
| Beginner | Common issues with straightforward diagnosis. |
| Intermediate | Multiple possible causes or several systems involved. |
| Advanced | Complex environments, uncommon failures, or deeper root cause analysis. |

Questions about tickets, contributions, or troubleshooting? Join the conversation in [GitHub Discussions](https://github.com/ryan-sharpnack/the-help-desk-project/discussions).
