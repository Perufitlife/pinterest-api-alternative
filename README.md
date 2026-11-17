# Pinterest API alternative: search results, profile pins, boards and pins as JSON, no login

How to get public Pinterest data as JSON without a Pinterest developer app, app approval or a logged-in account: keyword search results, the pins a profile created, every pin on a board and single pins by link, with titles, descriptions, image and video links, destination links, saves, reactions and dates.

The data comes from the Apify Actor [Pinterest Scraper](https://apify.com/renzomacar/pinterest-scraper). It reads pinterest.com the way a visitor without an account sees it. No cookies, and no proxy to set up.

## Why not the official Pinterest API

The official API is built around an account that authorizes your app. You register an app, wait for access to be approved, and then work mostly with the boards, pins and ad accounts of the people who signed in. That is the right tool for publishing pins and managing your own account. It is a slow road when the job is to read what ranks for a keyword or what other public accounts pin.

## Example: one search term, one profile, one board, one pin

Input:

```json
{
  "queries": ["home office ideas"],
  "maxPinsPerQuery": 60,
  "usernames": ["natgeo"],
  "maxPinsPerProfile": 40,
  "boardUrls": ["https://www.pinterest.com/natgeo/travel/"],
  "maxPinsPerBoard": 30,
  "pinUrls": ["https://www.pinterest.com/pin/146859637843480440/"]
}
```

Run on 2026-11-17: 131 rows in 5 seconds (60 search results, 40 profile pins, 30 board pins, 1 pin by link). Full file: [sample-output.json](sample-output.json).

Search results for "home office ideas" with the most reactions (`source: "search"`):

| Title | Destination site | Type | Reactions | Created | Author | Author followers |
|---|---|---|---|---|---|---|
| 12 Creative Home Office Built-Ins You Will Love | theolivebranchnest.com | idea | 76 | 2025-12-25 | @theolivebranchnest | 15568 |
| 6 Cozy Feminine Home Office Looks 2026 | dreambundles.com | idea | 11 | 2026-07-28 | @dreambundles | 866 |
| Dreamy White L-Shaped Built-In Study Corner / Ae… |  | idea | 10 | 2026-10-30 | @Pushparaj09 | 17 |
| Cozy Home Office Ideas / Warm & Stylish Workspac… |  | idea | 9 | 2026-10-09 | @trendy_homedecor | 69 |
| White and Oak minimalist Desk Setup | amazon.com | idea | 6 | 2026-11-03 | @deskfindsuk | 11 |

Latest pins created by @natgeo (`source: "profile"`):

| Created | Title | Type | Saves | Reactions | Comments | Board |
|---|---|---|---|---|---|---|
| 2025-12-23 | Give the Gift of Adventure! | video | 3 | 7 | 0 | Subscribe to National Geographic - Kids |
| 2025-12-23 | Explore the World Without Leaving Home! | video | 2 | 4 | 0 | Subscribe to National Geographic |
| 2025-06-17 | Stanley Tucci's guide to Tuscany's hidden gems | video | 3 | 4 | 0 | Travel |
| 2025-06-16 | What is noctourism? | image | 6 | 4 | 0 | Adventure |
| 2025-06-14 | How do seashells get their shapes? | image | 23 | 5 | 0 | Science |

Most saved pins on the board natgeo/travel (`source: "board"`):

| Title | Destination site | Saves | Reactions |
|---|---|---|---|
| Khasi root bridges in India |  | 23 | 8 |
| Discover the Croatian art of 'fjaka' | on.natgeo.com | 19 | 1 |
| Where's the best place to see the Northern Light… | on.natgeo.com | 19 | 1 |
| Where to see the spring blooms | on.natgeo.com | 18 | 3 |

One record:

```json
{
  "source": "search",
  "query": "home office ideas",
  "pinId": "e1_AfqPShzLSe9aivfXYFyGLPlonWAC6KRq0Bpkj7ZKaRo190MemXg",
  "url": "https://www.pinterest.com/pin/e1_AfqPShzLSe9aivfXYFyGLPlonWAC6KRq0Bpkj7ZKaRo190MemXg/",
  "title": "14 Office Craft Room Combo Ideas for Workdays and Creative Projects - MaterialSix",
  "description": "Long white counter divided into a computer station and a paper-crafting station beneath wall shelves.",
  "altText": "a home office with white shelves and plants on the desk, along with a wooden stool",
  "link": "https://materialsix.com/office-craft-room-combo/",
  "domain": "materialsix.com",
  "mediaType": "image",
  "imageUrl": "https://i.pinimg.com/originals/e5/e5/05/e5e505964c4fd532f8b37241e862a149.webp",
  "imageWidth": 683,
  "imageHeight": 1024,
  "thumbnailUrl": "https://i.pinimg.com/236x/e5/e5/05/e5e505964c4fd532f8b37241e862a149.jpg",
  "videoUrl": null,
  "saves": null,
  "reactions": 2,
  "comments": null,
  "createdAt": "2026-11-01T21:20:19.000Z",
  "username": "auntshel",
  "fullName": "Michele DiMattia",
  "authorFollowers": 58,
  "boardName": "Ed Nichols",
  "boardUrl": "https://www.pinterest.com/auntshel/ed-nichols/",
  "isPromoted": false,
  "dominantColor": "#977d64",
  "scrapedAt": "2026-11-17T10:26:24.464Z"
}
```

`source` says where a row came from: `search`, `profile`, `board` or `pin`. `mediaType` is `image`, `video`, `idea` or `carousel`.

## Call it from code

You need an Apify account and its API token (`APIFY_TOKEN`).

curl, results back in the same request:

```bash
curl -X POST "https://api.apify.com/v2/acts/renzomacar~pinterest-scraper/run-sync-get-dataset-items?token=$APIFY_TOKEN" \
  -H "content-type: application/json" \
  -d '{"queries":["home office ideas"],"maxPinsPerQuery":100}'
```

Python, which sites win a keyword:

```python
from collections import Counter
from apify_client import ApifyClient

client = ApifyClient("YOUR_APIFY_TOKEN")
run = client.actor("renzomacar/pinterest-scraper").call(run_input={
    "queries": ["home office ideas", "small home office", "desk setup"],
    "maxPinsPerQuery": 500,
})
rows = list(client.dataset(run["defaultDatasetId"]).iterate_items())
sites = Counter(r["domain"] for r in rows if r["domain"])
for site, pins in sites.most_common(20):
    print(pins, site)
```

JavaScript, the most saved pins of three profiles:

```js
import { ApifyClient } from "apify-client";

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });
const run = await client.actor("renzomacar/pinterest-scraper").call({
  usernames: ["natgeo", "etsy", "nike"],
  maxPinsPerProfile: 300,
});
const { items } = await client.dataset(run.defaultDatasetId).listItems();
items.sort((a, b) => (b.saves ?? 0) - (a.saves ?? 0));
for (const p of items.slice(0, 20)) console.log(p.saves, p.username, p.createdAt?.slice(0, 10), p.title, p.url);
```

## Use it from Claude, ChatGPT or another AI agent (MCP)

The Actor is available as a tool through the Apify MCP server. Add this to your MCP client configuration:

```json
{
  "mcpServers": {
    "apify": {
      "url": "https://mcp.apify.com/?tools=renzomacar/pinterest-scraper",
      "headers": { "Authorization": "Bearer YOUR_APIFY_TOKEN" }
    }
  }
}
```

Then ask in plain words, for example: "Search Pinterest for small balcony ideas and tell me which sites the top pins link to."

## Input fields

| Field | What it does |
|---|---|
| `queries` | Search terms, each searched separately. |
| `maxPinsPerQuery` | 1 to 10,000 per term. |
| `usernames` | Profiles: `natgeo` or a profile link. Returns the pins the profile created, newest first. |
| `maxPinsPerProfile` | 1 to 10,000 per profile. |
| `boardUrls` | Links to boards, such as `https://www.pinterest.com/natgeo/travel/`. |
| `maxPinsPerBoard` | 1 to 10,000 per board. |
| `pinUrls` | Links to single pins with a numeric ID. |

## What Pinterest shows without login

Measured on 2026-11-17, so you know what to expect:

| Source | Saves | Comments | Date | Reactions |
|---|---|---|---|---|
| `search` | no | no | yes | yes |
| `profile` | yes | yes | yes | yes |
| `board` | yes | no | no | yes |
| `pin` | yes | yes | yes | yes |

- Search: Pinterest stops at about 870 to 910 pins per term. To cover a subject, add several terms.
- Search and board rows carry a visitor pin ID that starts with `e1_`. The link opens in a browser, but Pinterest does not give the details of such a pin to a visitor.
- Profiles: you get the pins the profile created (550 for one large account, back to 2014). Pins saved from others live on the profile's boards.
- Boards were read in full in tests (81 of 81, 54 of 54, 41 of 43 pins).
- Private profiles and secret boards are not returned.

## Price

Pay per result on Apify: $1 per 1,000 pins. Nothing for starting a run, and a run that returns nothing costs nothing.
