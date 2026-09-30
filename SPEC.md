# HK Mahjong AI Score Calculator — Spec

`index.html` is a Hong Kong Mahjong (香港麻雀) score calculator. The user photographs
their winning hand and sets the win context the photo can't show. Claude vision
identifies the tiles, groups them into melds + pair, and returns the hand fan. The page
adds the win-context fan and shows the total fan and payout.

## AI scan

- Endpoint: `POST https://api.anthropic.com/v1/messages`, model `claude-sonnet-5-5`.
- The user pastes their own Anthropic API key in **Settings**. It is stored only in the
  browser's `localStorage` (`hkmj_anthropic_api_key`) and sent only to api.anthropic.com.
- Headers when a key is set: `x-api-key`, `anthropic-version: 2023-06-01`,
  `anthropic-dangerous-direct-browser-access: true`.
- With no key saved, the request is sent without auth headers (this only works inside
  Claude's artifact viewer); on an auth error the page asks for a key.
- **Never commit a real API key.**
- The AI scores only what's visible in the photo. Everything under "Win actions" and
  the three Blessings are manual toggles that the page adds itself.
- Hand fan = sum of the AI's `fan_items`.

## Fan table

### Win actions (manual toggles, NOT counted by the AI)

| Win action | 中文 | Fan |
|---|---|---|
| Self-Pick | 自摸 | +1 |
| Kong Replacement | 槓上開花 | +2 |
| Double Kong Replacement | 槓上槓 | +9 |
| Robbing the Kong | 搶槓 | +1 |
| Concealed Hand | 門前清 | +1 |
| Moon Under the Sea | 海底撈月 | +1 |

### Set type

| Hand | 中文 | Fan |
|---|---|---|
| All Sequences | 平糊 | 1 |
| All Triplets | 對對糊 | 3 |
| All Concealed Triplets | 坎坎糊 | 8 |
| All Quadruplets | 十八羅漢 | 13 |

### Honours

| Hand | 中文 | Fan |
|---|---|---|
| Dragon triplet | 三元牌 | +1 each |
| Small Three Dragons | 小三元 | 5 |
| Big Three Dragons | 大三元 | 8 |
| Round / Seat Wind triplet | 圈風 / 門風 | +1 (+2 if both) |
| Small Four Winds | 小四喜 | 6 |
| Big Four Winds | 大四喜 | 13 |

### Flush

| Hand | 中文 | Fan |
|---|---|---|
| Mixed Flush | 混一色 | 3 |
| Full Flush | 清一色 | 7 |
| Mixed Terminals | 么九 | 4 |
| All Terminals | 清么九 | 13 |
| All Honours | 字一色 | 10 |

### Special

| Hand | 中文 | Fan |
|---|---|---|
| Seven Pairs | 七對子 | 4 |
| Thirteen Orphans | 十三么 | 13 |
| Nine Gates | 九子連環 | 13 |
| Blessing of Heaven | 天糊 | 13 (manual toggle) |
| Blessing of Earth | 地糊 | 13 (manual toggle) |
| Blessing of Man | 人糊 | 13 (manual toggle) |

### Flowers

| Hand | 中文 | Fan |
|---|---|---|
| No Flowers | 無花 | +1 |
| Seat Flower / Season | 正花 | +1 each |
| All Flowers or All Seasons | 一台花 | +2 |
| Seven Flowers | 七枝花 | 3 |
| Eight Flowers | 八枝花 | 8 |

## Stacking

- Hands stack, e.g. All Triplets 3 + Full Flush 7 = 10.
- Full Flush replaces Mixed Flush.
- Big Three Dragons replaces Small Three Dragons and the individual dragon fan.
- Small Three Dragons replaces the individual dragon fan.
- Big Four Winds replaces Small Four Winds.
- 13+ fan = limit hand.

### Clarifications (from the HKMJ Rule Sheet v1.0, where the rules above are silent)

- Only one set type applies: All Concealed Triplets and All Quadruplets replace All Triplets.
- Small Four Winds and Big Four Winds replace the Round/Seat Wind fan.
- Mixed Terminals, All Terminals and All Honours already include the 3 fan for All
  Triplets, so All Triplets is not added on top. All Terminals and All Honours replace
  Mixed Terminals.
- Seven Pairs can stack with All Honours, Mixed Flush and Full Flush.
- Win actions are additive. For example, Self-Pick + Kong Replacement = 3.

## Payout

### New Style 新式 (discarder pays all 包底)

| Fan | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13+ |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| Points | 1 | 2 | 4 | 8 | 16 | 24 | 32 | 48 | 64 | 96 | 128 | 192 | 256 | 384 |

### Classical 古式

| Fan | 0 | 1 | 2 | 3 | 4–6 | 7–9 | 10–12 | 13+ |
|---|---|---|---|---|---|---|---|---|
| Points | 1 | 2 | 4 | 8 | 16 | 32 | 64 | 128 |

East wins/loses = all payments double. (If East wins, every payer pays ×2; if East pays, East's payment is ×2.)

### Shown on the result

- Non-discarder pays: points
- Discarder pays: points × 2
- Self-pick, each player pays: points × 2
