---
name: memeperfect-external-api
description: Use this skill when an agent needs to create, update, activate, inspect, or integrate MemePerfect Strategies through the public external API. This skill is restricted to documented external API capabilities and should not assume access to UI-only features.
license: MIT
compatibility: Requires HTTPS requests to the MemePerfect external API and a valid API key from the MemePerfect app.
metadata:
  author: MemePerfect
  version: "1.0"
---

## Capabilities

MemePerfect helps users build automated token discovery Strategies that combine on-chain security analysis, market metrics, social signals, and AI-powered narrative detection.

From an agent perspective, the important distinction is:

- the product has broader UI capabilities and workflows
- this skill only authorizes actions that are documented in the public external API

MemePerfect exposes a public external API for:

- reading authenticated account limits
- creating and listing Strategies
- updating, activating, deactivating, and deleting Strategies
- reading notifications and performance summaries
- configuring outbound webhooks

Agents should treat this file as the source of truth for what is supported through the public API.

## Product context

Use this context to reason about user intent and payload design:

- `Strategy` is the main object that combines triggers, rules, and delivery behavior
- `Trigger` defines when a Strategy runs
- `Dealbreakers` are hard-fail checks that block alerts before grouped rules are evaluated
- `Rules` are the conditional checks inside rule groups
- `Flexible Matching` allows partial rule matches when enabled
- `Backtests`, `Watchlist`, `Twitter Handles`, `Dev Tracking`, and `AI Narrative Engine` are product concepts that may appear in docs and user requests

This context helps the agent understand the platform, but it does not expand the public API surface.

## Skills

These describe what the product is designed to do. Only use them as reasoning context unless a matching external API endpoint is documented below.

### Strategy building and automation

**Create custom Strategies**: Build multi-layered token discovery Strategies that combine event-driven Triggers with rule-based evaluation.

**Flexible Matching**: Use percentage-based matching to allow a Strategy to pass when a defined portion of grouped rules are satisfied.

**Strict mode**: Require more complete input data during evaluation.

**Cooldown control**: Suppress repeated alerts for the same token with `alertCooldownMins`.

### Trigger types

These concepts map to documented `eventType` values:

**New token created**: Evaluate newly launched tokens.

**Token graduation events**: Evaluate tokens reaching later lifecycle states.

**Tweet mentions**: Evaluate tokens when tracked tweet data matches direct mentions.

**AI-generated narratives**: Evaluate tokens against narrative-derived matches from tweet metadata.

**Developer tracking**: Evaluate tokens launched by followed developers when that trigger is used.

### Evaluation dimensions

Strategies usually evaluate across four fixed rule-group dimensions:

**Security**: Contract safety, ownership concentration, creator history, LP state, token age.

**Market**: Liquidity, market cap, taxes, trading activity, holder count.

**Socials**: Website, Telegram, Twitter/X, social presence, link-content checks.

**AI**: Content validation scores derived from website or X content.

## Inputs required

To act successfully, an agent needs:

- `base_url`: `https://api.memeperfect.io/api/external/v1`
- `api_key`: sent as `X-API-Key` or `Authorization: ApiKey ...`
- a valid Strategy payload when creating or updating a Strategy
- a webhook URL when configuring webhook delivery

## Authentication and limits

Use one of these headers:

```http
X-API-Key: mpk_your_api_key_here
```

```http
Authorization: ApiKey mpk_your_api_key_here
```

Check limits with `GET /me`.

Typical plan limits:

- `PRO`: 60 requests per minute, up to 25 active Strategies
- `DEGEN`: 300 requests per minute, up to 50 active Strategies

## Supported endpoints

Use only these documented endpoint families:

- `GET /me`
- `GET /strategies`
- `POST /strategies`
- `GET /strategies/:id`
- `PATCH /strategies/:id`
- `POST /strategies/:id/activate`
- `POST /strategies/:id/deactivate`
- `DELETE /strategies/:id`
- `GET /notifications`
- `GET /notifications/:id`
- `GET /notifications/performance`
- `POST /notifications/performance/jobs`
- `GET /notifications/performance/jobs/:jobId`
- `GET /webhook`
- `PUT /webhook`

## Strategy payload rules

When creating or updating Strategies, preserve these product terms exactly:

- `Strategy`
- `Trigger`
- `Rules`
- `Dealbreakers`
- `Flexible Matching`
- `Backtests`
- `Watchlist`
- `Twitter Handles`
- `Dev Tracking`
- `AI Narrative Engine`

For API payloads:

- `triggers`, `dealbreakers`, `ruleGroups`, and `isActive` are core Strategy fields
- `strict` is optional and controls data strictness
- `alertCooldownMins` is optional and suppresses repeat alerts for the same token within the cooldown window
- `matching.enabled` and `matching.minPercent` control Flexible Matching behavior

## Strategy payload format

Use this as the canonical create/update shape:

```json
{
  "name": "API Strategy Example",
  "description": "Programmatic strategy",
  "strict": true,
  "alertCooldownMins": 30,
  "matching": {
    "enabled": true,
    "minPercent": 80
  },
  "triggers": [
    {
      "id": "trigger-1",
      "eventType": "new_token_created",
      "enabled": true
    }
  ],
  "dealbreakers": [
    {
      "id": "db-1",
      "ruleType": "is_honeypot",
      "operator": "is",
      "value": false,
      "enabled": true
    }
  ],
  "ruleGroups": [
    {
      "id": "security",
      "name": "Security",
      "logic": "AND",
      "rules": []
    },
    {
      "id": "market",
      "name": "Market",
      "logic": "AND",
      "rules": []
    },
    {
      "id": "socials",
      "name": "Socials",
      "logic": "AND",
      "rules": []
    },
    {
      "id": "ai",
      "name": "AI",
      "logic": "AND",
      "rules": []
    }
  ],
  "isActive": false
}
```

Field notes:

- `name`: required string
- `description`: optional string
- `strict`: optional boolean
- `alertCooldownMins`: optional number from `0` to `1440`
- `matching.enabled`: optional boolean
- `matching.minPercent`: required when matching is enabled
- `triggers`: required array
- `dealbreakers`: required array
- `ruleGroups`: required array using the four canonical groups
- `isActive`: required boolean on create

## Canonical request and response examples

Use these examples as the default reference shapes.

### `GET /me`

Request:

```bash
curl -X GET 'https://api.memeperfect.io/api/external/v1/me' \
  -H 'X-API-Key: YOUR_API_KEY'
```

Response:

```json
{
  "userId": "4fcb13dc-2f0c-49bb-97a3-2f2f7ac0f80f",
  "plan": "PRO",
  "limits": {
    "strategies": {
      "total": null,
      "active": 25
    },
    "rateLimit": {
      "requestsPerMinute": 60
    }
  }
}
```

### `POST /strategies`

Request:

```bash
curl -X POST 'https://api.memeperfect.io/api/external/v1/strategies' \
  -H 'X-API-Key: YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "API Safe Launch Flow",
    "description": "Programmatic strategy for safer early launches",
    "strict": true,
    "alertCooldownMins": 30,
    "triggers": [
      {
        "id": "trigger-new-token",
        "eventType": "new_token_created",
        "enabled": true
      }
    ],
    "dealbreakers": [
      {
        "id": "db-honeypot",
        "ruleType": "is_honeypot",
        "operator": "is",
        "value": false,
        "enabled": true
      }
    ],
    "ruleGroups": [
      {
        "id": "security",
        "name": "Security",
        "logic": "AND",
        "rules": [
          {
            "id": "r-risk",
            "ruleType": "risk_level",
            "operator": "not_equals",
            "value": "CRITICAL",
            "enabled": true
          }
        ]
      },
      {
        "id": "market",
        "name": "Market",
        "logic": "AND",
        "rules": [
          {
            "id": "r-liquidity-min",
            "ruleType": "liquidity_usd",
            "operator": "greater_than",
            "value": 10000,
            "enabled": true
          }
        ]
      },
      {
        "id": "socials",
        "name": "Socials",
        "logic": "AND",
        "rules": [
          {
            "id": "r-website",
            "ruleType": "has_website",
            "operator": "is",
            "value": true,
            "enabled": true
          }
        ]
      },
      {
        "id": "ai",
        "name": "AI",
        "logic": "AND",
        "rules": []
      }
    ],
    "matching": {
      "enabled": true,
      "minPercent": 80
    },
    "isActive": false
  }'
```

Response:

```json
{
  "strategy": {
    "id": "9a46ddf3-98fd-4cf5-8fd0-022421741c35",
    "name": "API Safe Launch Flow",
    "description": "Programmatic strategy for safer early launches",
    "userId": "4fcb13dc-2f0c-49bb-97a3-2f2f7ac0f80f",
    "triggers": [
      {
        "id": "trigger-new-token",
        "eventType": "new_token_created",
        "enabled": true
      }
    ],
    "dealbreakers": [
      {
        "id": "db-honeypot",
        "ruleType": "is_honeypot",
        "operator": "is",
        "value": false,
        "enabled": true
      }
    ],
    "ruleGroups": [
      {
        "id": "security",
        "name": "Security",
        "logic": "AND",
        "rules": [
          {
            "id": "r-risk",
            "ruleType": "risk_level",
            "operator": "not_equals",
            "value": "CRITICAL",
            "enabled": true
          }
        ]
      },
      {
        "id": "market",
        "name": "Market",
        "logic": "AND",
        "rules": [
          {
            "id": "r-liquidity-min",
            "ruleType": "liquidity_usd",
            "operator": "greater_than",
            "value": 10000,
            "enabled": true
          }
        ]
      },
      {
        "id": "socials",
        "name": "Socials",
        "logic": "AND",
        "rules": [
          {
            "id": "r-website",
            "ruleType": "has_website",
            "operator": "is",
            "value": true,
            "enabled": true
          }
        ]
      },
      {
        "id": "ai",
        "name": "AI",
        "logic": "AND",
        "rules": []
      }
    ],
    "strict": true,
    "alertCooldownMins": 30,
    "isActive": false,
    "isGlobal": false,
    "matching": {
      "enabled": true,
      "minPercent": 80
    },
    "createdAt": "2026-03-09T08:25:58.221Z",
    "updatedAt": "2026-03-09T08:25:58.221Z"
  }
}
```

### `PATCH /strategies/:id`

Request:

```bash
curl -X PATCH 'https://api.memeperfect.io/api/external/v1/strategies/9a46ddf3-98fd-4cf5-8fd0-022421741c35' \
  -H 'X-API-Key: YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "alertCooldownMins": 45,
    "ruleGroups": [
      {
        "id": "security",
        "name": "Security",
        "logic": "AND",
        "rules": []
      },
      {
        "id": "market",
        "name": "Market",
        "logic": "AND",
        "rules": [
          {
            "id": "r-liquidity-min",
            "ruleType": "liquidity_usd",
            "operator": "greater_than",
            "value": 15000,
            "enabled": true
          }
        ]
      },
      {
        "id": "socials",
        "name": "Socials",
        "logic": "AND",
        "rules": []
      },
      {
        "id": "ai",
        "name": "AI",
        "logic": "AND",
        "rules": []
      }
    ]
  }'
```

Response:

```json
{
  "strategy": {
    "id": "9a46ddf3-98fd-4cf5-8fd0-022421741c35",
    "alertCooldownMins": 45,
    "updatedAt": "2026-03-09T09:10:18.401Z"
  }
}
```

### `GET /notifications`

Request:

```bash
curl -X GET 'https://api.memeperfect.io/api/external/v1/notifications?page=1&limit=20&strategyId=9a46ddf3-98fd-4cf5-8fd0-022421741c35' \
  -H 'X-API-Key: YOUR_API_KEY'
```

Response:

```json
{
  "items": [
    {
      "id": "497e3b40-3a6e-449f-87cd-4f96af15cffe",
      "userId": "4fcb13dc-2f0c-49bb-97a3-2f2f7ac0f80f",
      "strategyId": "9a46ddf3-98fd-4cf5-8fd0-022421741c35",
      "channel": "telegram",
      "status": "sent",
      "triggerEventType": "new_token_created",
      "triggerContext": {
        "tokenAddress": "7xKXExampleAddress",
        "tokenData": {
          "tokenSymbol": "EXAMPLE",
          "market": {
            "priceUSD": 0.00012,
            "mcap": 120000
          }
        }
      },
      "sentAt": "2026-03-09T08:31:05.120Z",
      "createdAt": "2026-03-09T08:31:05.120Z",
      "updatedAt": "2026-03-09T08:31:05.120Z",
      "strategy": {
        "id": "9a46ddf3-98fd-4cf5-8fd0-022421741c35",
        "name": "API Safe Launch Flow"
      }
    }
  ],
  "page": 1,
  "limit": 20,
  "total": 1,
  "totalPages": 1
}
```

### `GET /notifications/performance`

Request:

```bash
curl -X GET 'https://api.memeperfect.io/api/external/v1/notifications/performance?range=daily&strategyId=9a46ddf3-98fd-4cf5-8fd0-022421741c35' \
  -H 'X-API-Key: YOUR_API_KEY'
```

Response:

```json
{
  "range": "daily",
  "summary": {
    "notifications": 12,
    "wins": 4,
    "losses": 3,
    "pending": 5,
    "winRate": 57.14
  }
}
```

### `PUT /webhook`

Request:

```bash
curl -X PUT 'https://api.memeperfect.io/api/external/v1/webhook' \
  -H 'X-API-Key: YOUR_API_KEY' \
  -H 'Content-Type: application/json' \
  -d '{
    "enabled": true,
    "url": "https://your-domain.com/memeperfect/webhook"
  }'
```

Response:

```json
{
  "enabled": true,
  "url": "https://your-domain.com/memeperfect/webhook",
  "verifiedAt": "2026-03-09T08:36:21.522Z",
  "hasSecret": true,
  "secret": "whsec_9SNQ...",
  "verificationError": null
}
```

## Fixed rule group contract

For UI compatibility, agents must not invent arbitrary rule groups.

Use exactly these four rule groups:

- `security` with name `Security`
- `market` with name `Market`
- `socials` with name `Socials`
- `ai` with name `AI`

Each group must include:

- `id`
- `name`
- `logic` as `AND` or `OR`
- `rules` as an array

## Allowed trigger eventType values

Use only these documented `eventType` values:

- `new_token_created`
- `followed_dev_new_token_created`
- `twitter_mention_direct`
- `tweet_metadata_match`
- `token_almost_graduated`
- `token_graduated`

## Rule object contract

Each rule in `dealbreakers` or `ruleGroups[].rules` uses this shape:

```json
{
  "id": "rule-1",
  "ruleType": "liquidity_usd",
  "operator": "greater_than",
  "value": 10000,
  "enabled": true
}
```

Allowed operators:

- `greater_than`
- `less_than`
- `equals`
- `not_equals`
- `is`
- `is_not`

Agents must pair operators with compatible rule types and values. Do not guess unsupported operator and value combinations.

## Rule catalog

Use only documented `ruleType` values. Group them under the four canonical rule groups.

### Security ruleTypes

- `is_honeypot`
- `is_mintable`
- `top_10_holder_percent`
- `dev_wallet_percent`
- `is_lp_locked`
- `risk_level`
- `rugged`
- `lp_locked_pct`
- `has_freeze_authority`
- `has_high_ownership`
- `has_top10_high_ownership`
- `has_single_holder_ownership`
- `has_creator_rug_history`
- `has_low_lp_providers`
- `has_low_liquidity`
- `creator_has_multiple_tokens`
- `has_creator_balance`
- `has_lockers`
- `token_mutable`
- `has_transfer_fee`
- `token_age`

### Market ruleTypes

- `sell_tax`
- `buy_tax`
- `liquidity_usd`
- `market_cap`
- `volume_24h`
- `tx_count`
- `num_buys`
- `num_sells`
- `snipers`
- `insiders`
- `bundle`
- `bonding_progress`
- `is_pump`
- `holder_count`

### Socials ruleTypes

- `has_website`
- `has_telegram`
- `has_twitter`
- `at_least_one_social`
- `website_has_ca`
- `website_contains_token_name`
- `twitter_link_has_ca`
- `x_link_contains_token_name`
- `twitter_effective_views`
- `x_community_member_count`
- `x_community_views_count`
- `twitter_effective_followers`

### AI ruleTypes

- `website_content_validation_score`
- `x_content_validation_score`

## Rule value guidance

Use these operator and value patterns:

- boolean rules usually use `is` or `is_not` with `true` or `false`
- numeric and percentage rules use `greater_than` or `less_than` with numeric values
- string rules such as `risk_level` use `equals` or `not_equals`

Known enum values:

- `risk_level`: `CRITICAL`, `HIGH`, `MEDIUM`, `LOW`

AI preset guidance:

- `very_relevant` maps to score range rules `greater_than: 80` and `less_than: 100`
- `somewhat_relevant` maps to score range rules `greater_than: 50` and `less_than: 100`
- `any` means omit those AI score rules

## Workflow

Use this order for most integrations:

1. Call `GET /me` to confirm authentication, plan, and limits.
2. Build a valid Strategy payload using the fixed rule groups and allowed trigger values.
3. Call `POST /strategies` to create the Strategy.
4. Call `POST /strategies/:id/activate` to make it live.
5. Call `GET /notifications` or `GET /notifications/:id` to read outcomes.
6. Call `GET /notifications/performance` or async performance job endpoints for results analysis.
7. Call `PUT /webhook` if the user wants programmatic delivery.
8. Call `PATCH /strategies/:id` or `POST /strategies/:id/deactivate` when changing or disabling behavior.

## Agent workflow modes

Use these modes when deciding how aggressively to act.

### Careful mode

1. Read `GET /me`.
2. Show the proposed Strategy payload to the human before creation.
3. Create with `isActive: false`.
4. Wait for approval before activation.

### Standard mode

1. Read `GET /me`.
2. Create a valid Strategy from the user request.
3. Activate if the user explicitly asked for a live Strategy.
4. Configure webhook only if the user provided an endpoint.

### Maintenance mode

1. Fetch the existing Strategy.
2. Update only the requested fields.
3. Preserve fixed group IDs and names.
4. Re-check notifications or performance after the update if the user asked for validation.

## Webhook behavior

Webhook configuration is available through the external API.

Important constraints:

- setting or changing the webhook URL triggers endpoint verification
- webhook signing uses `X-MP-Signature`
- verification uses a challenge-response flow
- webhook delivery should be treated as best-effort

## Errors

Expect at least these status codes:

- `400` invalid body, query, or path parameters
- `401` missing or invalid API key
- `403` subscription required or active Strategy limit reached
- `404` resource not found or not owned by the authenticated user
- `429` rate limit exceeded

Common error shape:

```json
{
  "statusCode": 401,
  "message": "API key required. Provide via X-API-Key header or Authorization: ApiKey <key>",
  "error": "Unauthorized"
}
```

## Do not assume

Agents must not claim or attempt undocumented public API capabilities.

Do not assume public API support for:

- template cloning
- backtest creation or execution
- Watchlist management
- Twitter Handles management
- Dev Tracking list management
- Telegram settings changes
- UI wizard actions
- internal admin endpoints

If a user asks for one of these, explain that it is not documented in the public external API and point them to the main docs for product guidance.

## Common mistakes to avoid

- Do not use arbitrary rule group names or IDs.
- Do not describe UI actions as API actions.
- Do not use undocumented trigger names.
- Do not omit `alertCooldownMins` when documenting full Strategy behavior.
- Do not say all rules always need to pass; account for `Dealbreakers`, per-group `logic`, and `Flexible Matching`.
- Do not expose API keys in browser code.

## Reference docs

Use these docs for deeper context:

- `https://docs.memeperfect.io/llms.txt`
- `https://docs.memeperfect.io/developers/apis`
- `https://docs.memeperfect.io/developers/api-walkthrough`
- `https://docs.memeperfect.io/developers/strategy-rules`
- `https://docs.memeperfect.io/developers/webhooks`
