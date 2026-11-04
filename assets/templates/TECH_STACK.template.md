# Tech Stack — <Project Name>

> **Purpose:** record what we build with and why.
> **Owner:** <name> · **Last reviewed:** <date>

## 1. At a Glance

| Concern | Choice | Version |
|---------|--------|---------|
| Language | <language> | <version> |
| Framework | <framework> | <version> |
| Package manager | <tool> | <version> |
| Testing | <tool> | <version> |

## 2. Core Choices

For each significant choice, state the reason and the alternative you rejected.

### <Choice 1>

- **Use it because:** <reason>
- **Rejected alternative:** <alternative> — because <reason>

### <Choice 2>

- **Use it because:** <reason>
- **Rejected alternative:** <alternative> — because <reason>

## 3. Dependencies

| Package | Purpose | Runtime or dev |
|---------|---------|----------------|
| <name> | <purpose> | runtime |

## 4. External Services

| Service | Used for | Auth | Limits / cost |
|---------|----------|------|---------------|
| <service> | <purpose> | <method> | <notes> |

## 5. Runtime & Environment

- **Runs on:** <OS / container / cloud>
- **Process model:** <single process / workers / serverless>
- **State:** <ephemeral / persistent, and where>

## 6. Explicitly Not Used

| Thing | Why not (for v1) |
|-------|------------------|
| <tech> | <reason> |

This section prevents scope creep; update it when the decision changes.

## Exit checklist

- [ ] Every major choice has a rejected alternative
- [ ] Versions are pinned
- [ ] Secrets and config are env-only
- [ ] The "not used" list is honest and current
