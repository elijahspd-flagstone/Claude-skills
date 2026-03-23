# Analytics Events Template — Codepup Feature Breakdown

## Naming Convention

**Format:** `[noun]_[past_tense_verb]`

Always snake_case. Always past tense. Always specific enough that you know what fired
without reading the code.

Good: `site_published`, `template_selected`, `plan_upgraded`
Bad: `click`, `success`, `feature_used`, `button_clicked`

---

## Required Events for Every Feature

Every feature MUST have these four lifecycle events at minimum.

| Event | When it fires | Required properties |
|-------|--------------|---------------------|
| `[feature]_viewed` | User sees the feature surface for the first time in a session | `user_id`, `session_id`, `plan_tier`, `source` |
| `[feature]_started` | User takes the first meaningful action | `user_id`, `session_id`, `entry_point` |
| `[feature]_completed` | User reaches the terminal success state | `user_id`, `session_id`, `duration_seconds`, `steps_taken` |
| `[feature]_abandoned` | User exits without completing | `user_id`, `session_id`, `last_step`, `time_in_flow_seconds` |

Replace `[feature]` with the feature noun. Examples:
- `checkout_viewed`, `checkout_started`, `checkout_completed`, `checkout_abandoned`
- `site_builder_viewed`, `site_builder_started`, `site_builder_completed`, `site_builder_abandoned`

---

## Standard Property Reference

Use these property names consistently across all events. Never invent synonyms.

### User Properties
| Property | Type | Description |
|----------|------|-------------|
| `user_id` | string | Codepup internal user ID (never email) |
| `plan_tier` | enum | `free`, `starter`, `pro`, `enterprise` |
| `account_age_days` | integer | Days since user signed up |
| `is_onboarding` | boolean | True if within first 7 days |

### Session Properties
| Property | Type | Description |
|----------|------|-------------|
| `session_id` | string | Current session identifier |
| `source` | enum | `direct`, `email`, `organic`, `paid`, `referral` |
| `device_type` | enum | `desktop`, `tablet`, `mobile` |

### Flow Properties
| Property | Type | Description |
|----------|------|-------------|
| `entry_point` | string | Where the user came from (e.g., `dashboard`, `sidebar`, `onboarding`) |
| `duration_seconds` | integer | Time from started to completed |
| `steps_taken` | integer | Number of user actions taken |
| `last_step` | string | Name of the last step reached before abandon |

### Error Properties
| Property | Type | Description |
|----------|------|-------------|
| `error_code` | string | Machine-readable error identifier |
| `error_message` | string | Human-readable message shown to user |
| `step` | string | Which step in the flow the error occurred |

---

## Common Feature-Specific Events

Adapt these patterns for your feature:

### Content Creation Features
```
[content_type]_created      — user creates a new item
[content_type]_edited       — user modifies an existing item
[content_type]_deleted      — user removes an item
[content_type]_published    — user makes an item live/visible
[content_type]_duplicated   — user copies an existing item
```

### Configuration / Settings Features
```
settings_opened             — user opens a settings panel
settings_saved              — user confirms a settings change
settings_reset              — user reverts to defaults
```

### Integration / Connection Features
```
integration_connected       — user successfully links a third-party
integration_disconnected    — user unlinks a third-party
integration_auth_failed     — OAuth or API key auth failed
integration_synced          — data sync completed successfully
```

### Paywall / Upgrade Features
```
paywall_hit                 — user tries a feature they can't access
upgrade_modal_viewed        — upgrade prompt shown
upgrade_cta_clicked         — user clicks upgrade button
plan_upgraded               — payment confirmed, tier changed
```

### Search / Discovery Features
```
search_initiated            — user types in a search box
search_completed            — results returned
result_selected             — user clicks a search result
filter_applied              — user narrows results
```

---

## What NOT to Track

- PII: email, name, phone, address — never in event properties
- Payment details: card numbers, CVV, full amounts
- Passwords or tokens
- Free-text fields that users type into (capture metadata, not content)

---

## Validation Checklist Before Shipping

- [ ] All four lifecycle events (viewed/started/completed/abandoned) are instrumented
- [ ] Each event fires exactly once per user action (no duplicates)
- [ ] All properties use the standard names from this template
- [ ] No PII in any property
- [ ] Events appear in Amplitude within 60s in a local test
- [ ] Abandoned event fires on browser close, navigate-away, and timeout
- [ ] Event names are added to the Amplitude schema in advance of release
