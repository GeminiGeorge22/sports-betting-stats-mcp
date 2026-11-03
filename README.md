# Sports Betting Stats MCP: odds, player props, scores, xG, horse racing, prediction markets

**Everything a betting model or sports assistant needs, as MCP tools**: live odds from US and European sportsbooks, player props, scores and schedules, football xG and shot data, historical closing lines, US horse racing entries and results, and prediction-market prices. One hosted endpoint, one Actor per job.

Hosted remote MCP server (Streamable HTTP) on [Apify's MCP gateway](https://mcp.apify.com). **Client config plus a small open-source stdio proxy server. No Actor source.**

**Disclosure:** entries marked "(maintained by us)" are Actors I maintain on Apify Store (publisher `publicrecords`). Every other Actor is independent, made by its own publisher and picked on merit (Store usage, ratings and recent maintenance). This repo is not affiliated with those publishers or with the sites the Actors read.

## MCP endpoint

```
https://mcp.apify.com/?tools=publicrecords/sports-odds-api,scrapemint/sports-odds-scraper,zen-studio/prizepicks-player-props,sports-odds-lab/football-odds-results,hgservices/apify-actor-espn,constructive_calm/understat-football-analytics,jungle_synthesizer/equibase-us-horse-racing-scraper,congism/polymarket-kalshi-arb-finder
```

Auth: sign in with Apify (OAuth) when your client prompts, or send an Apify API token as a Bearer token. Runs are billed on your own Apify account under each Actor's own Store pricing; see each Store page.

## Tools

| Actor | Store page | MCP tool | What it returns |
|---|---|---|---|
| **Sports Odds API (Kambi books)** (maintained by us) | [publicrecords/sports-odds-api](https://apify.com/publicrecords/sports-odds-api) | `publicrecords--sports-odds-api` | Live odds from Kambi-powered sportsbooks (BetRivers, Unibet, Paf, Svenska Spel, ATG): one row per game with every book's price and the best price per outcome. |
| **Sports Odds Scraper** | [scrapemint/sports-odds-scraper](https://apify.com/scrapemint/sports-odds-scraper) | `scrapemint--sports-odds-scraper` | DraftKings, Pinnacle and Bovada moneyline, spread and totals on one line per game, with best price and arbitrage. |
| **PrizePicks Player Props** | [zen-studio/prizepicks-player-props](https://apify.com/zen-studio/prizepicks-player-props) | `zen-studio--prizepicks-player-props` | Real-time PrizePicks player prop lines across many leagues: over/under lines, odds tiers, matchups. |
| **Football Odds & Results** | [sports-odds-lab/football-odds-results](https://apify.com/sports-odds-lab/football-odds-results) | `sports-odds-lab--football-odds-results` | Soccer odds with opening and closing lines from many bookmakers, plus results, for 1,000+ leagues. |
| **ESPN Scores & Schedules** | [hgservices/apify-actor-espn](https://apify.com/hgservices/apify-actor-espn) | `hgservices--apify-actor-espn` | Live scores, schedules and results from ESPN: NBA, NFL, MLB, NHL, MLS, EPL, NCAA and more. |
| **Understat xG & Shot Data** | [constructive_calm/understat-football-analytics](https://apify.com/constructive_calm/understat-football-analytics) | `constructive_calm--understat-football-analytics` | Understat player and team xG, per-match win probabilities and shot-level data for the top European leagues. |
| **Equibase Horse Racing** | [jungle_synthesizer/equibase-us-horse-racing-scraper](https://apify.com/jungle_synthesizer/equibase-us-horse-racing-scraper) | `jungle_synthesizer--equibase-us-horse-racing-scraper` | US Thoroughbred race entries (horses, jockeys, trainers, odds) and results (finish order, times, payouts). |
| **Polymarket + Kalshi Arbitrage Finder** | [congism/polymarket-kalshi-arb-finder](https://apify.com/congism/polymarket-kalshi-arb-finder) | `congism--polymarket-kalshi-arb-finder` | Matches binary markets on Polymarket and Kalshi and ranks price gaps between them. |

The Apify gateway also exposes helper tools to read run results (`get-actor-run`, `get-dataset-items`). Want fewer tools? Drop any Actor from the `tools=` list.

## Who it's for

- Bettors and tipsters: who has the best price on this game right now?
- Modelers: closing lines, results and xG for backtests.
- DFS and props players: today's PrizePicks lines next to the scores.
- Racing fans: today's Equibase entries and yesterday's payouts.

## Example prompts

- Compare tonight's NBA moneylines across BetRivers, Unibet, DraftKings and Pinnacle and show the best price per side.
- Which PrizePicks NBA props are posted for `<player>` today?
- Give me the opening and closing odds and the result for `<team>`'s last 10 Premier League matches.
- Show `<team>`'s xG for and against over its last five matches.

## Cursor / Claude Code / any remote-MCP client

```json
{
  "mcpServers": {
    "sports-betting-stats": {
      "url": "https://mcp.apify.com/?tools=publicrecords/sports-odds-api,scrapemint/sports-odds-scraper,zen-studio/prizepicks-player-props,sports-odds-lab/football-odds-results,hgservices/apify-actor-espn,constructive_calm/understat-football-analytics,jungle_synthesizer/equibase-us-horse-racing-scraper,congism/polymarket-kalshi-arb-finder"
    }
  }
}
```

Claude Code:

```sh
claude mcp add --transport http sports-betting-stats "https://mcp.apify.com/?tools=publicrecords/sports-odds-api,scrapemint/sports-odds-scraper,zen-studio/prizepicks-player-props,sports-odds-lab/football-odds-results,hgservices/apify-actor-espn,constructive_calm/understat-football-analytics,jungle_synthesizer/equibase-us-horse-racing-scraper,congism/polymarket-kalshi-arb-finder"
```

## Claude Desktop (local bridge)

```json
{
  "mcpServers": {
    "sports-betting-stats": {
      "command": "npx",
      "args": ["-y", "@apify/actors-mcp-server", "--actors", "publicrecords/sports-odds-api,scrapemint/sports-odds-scraper,zen-studio/prizepicks-player-props,sports-odds-lab/football-odds-results,hgservices/apify-actor-espn,constructive_calm/understat-football-analytics,jungle_synthesizer/equibase-us-horse-racing-scraper,congism/polymarket-kalshi-arb-finder"],
      "env": { "APIFY_TOKEN": "YOUR_APIFY_TOKEN" }
    }
  }
}
```

## Local stdio server (open source)

This repo also ships a small MCP server in [`src/`](src/) for clients that only run local stdio servers. It answers `initialize` and `tools/list` locally from [`tools.json`](tools.json), which is generated from each Actor's public input schema and checked for drift in CI and weekly, so any client or directory can inspect the tools with no token. `tools/call` is forwarded to the hosted Apify MCP endpoint above with **your own** `APIFY_TOKEN`; without a token a call returns a clear error instead of running anything. The token is only sent in the `Authorization` header and is never logged.

```json
{
  "mcpServers": {
    "sports-betting-stats": {
      "command": "npx",
      "args": ["-y", "github:GeminiGeorge22/sports-betting-stats-mcp"],
      "env": { "APIFY_TOKEN": "YOUR_APIFY_TOKEN" }
    }
  }
}
```

Or from a clone:

```bash
git clone https://github.com/GeminiGeorge22/sports-betting-stats-mcp.git
cd sports-betting-stats-mcp && npm install
APIFY_TOKEN=YOUR_APIFY_TOKEN node src/server.js
```

Optional env: `APIFY_ACTORS` (comma-separated subset of the Actors above) and `APIFY_MCP_URL` (full endpoint override).

Tests run against an in-memory mock server, with no network and no token: `npm test`. Requires Node.js 18+.

## See also

- [ecommerce-search-rankings-mcp](https://github.com/GeminiGeorge22/ecommerce-search-rankings-mcp): E-commerce search results and rankings
- [ecommerce-sales-mcp](https://github.com/GeminiGeorge22/ecommerce-sales-mcp): E-commerce sales signals
- [forecast-accuracy-mcp](https://github.com/GeminiGeorge22/forecast-accuracy-mcp): Forecast accuracy and track records
- [sports-odds-mcp](https://github.com/GeminiGeorge22/sports-odds-mcp): Kambi sportsbook odds
- [etsy-sales-mcp](https://github.com/GeminiGeorge22/etsy-sales-mcp): Etsy search and shop sales
- [ats-jobs-mcp](https://github.com/GeminiGeorge22/ats-jobs-mcp): ATS job postings (Greenhouse, Lever, Ashby, SmartRecruiters)
- [gov-tenders-mcp](https://github.com/GeminiGeorge22/gov-tenders-mcp): Government tenders and grants

## Registry

Official MCP Registry name: `io.github.GeminiGeorge22/sports-betting-stats-mcp`

## License

MIT, for this documentation, config and the proxy server code in `src/`. Each Actor belongs to its publisher.
