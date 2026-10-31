# carris-metropolitana-mcp

<!-- mcp-name: io.github.ThiagoJacobLannes/carris-metropolitana-mcp -->

MCP server for [Carris Metropolitana](https://www.carrismetropolitana.pt/), the bus network of Greater Lisbon (Lisbon Metropolitan Area), so Claude (or any MCP client) can answer "when is the next bus at X?", "where does line 4715 go?" or "is my line disrupted?" in real time.

**Read-only, no key, no account.** It only reads the official open API. Not affiliated with Carris Metropolitana or TML (Transportes Metropolitanos de Lisboa).

Note: buses inside the city of Lisbon are mostly run by Carris, a different company that this server does not cover. Carris Metropolitana serves the surrounding municipalities and the routes into Lisbon (e.g. from Oriente, Cacilhas, Setúbal).

## Tools

### `find_stops`
Find stops by name and/or near a coordinate. Use it first to get a `stop_id`.

Parameters:
- `query` — text in the stop name, accent/case-insensitive (e.g. `"Gare do Oriente"`, `"cacilhas terminal"`); a 6-digit stop id also works
- `lat` / `lon` — a coordinate; results are sorted by straight-line distance
- `radius_m` — only stops within this radius (default 500 m when no `query` is given)
- `limit` — max stops (default 10, max 50)

Each stop has: id, name, locality, municipality, distance, coordinates, the lines serving it, and wheelchair boarding.

### `next_arrivals`
Upcoming buses at a stop, with minutes from now. Each arrival is marked `"realtime"` (live GPS estimate, with delay vs. timetable) or `"scheduled"` (timetable only).

Parameters:
- `stop_id` — 6-digit stop id (e.g. `"060361"`)
- `line_id` — optional, keep only one line (e.g. `"2794"`)
- `limit` — max arrivals (default 10, max 50)

### `line_info`
A line's name, municipalities and localities served, and each direction (headsign, first and last stop, number of stops, trips today, first/last departure today). Pass a line number (`"4715"`) or text (`"Oriente Setubal"`), in which case matching lines are listed. `include_stops=True` adds the ordered stop list for each direction.

### `service_alerts`
Active (and, by default, upcoming) service alerts: detours, works, strikes, stops not served. Filter by `line_id` and/or `stop_id`. Alerts are mostly published in Portuguese.

### `vehicle_positions`
Live positions of the buses currently running on a line: coordinates, heading, speed, headsign, the stop it is at or heading to, and how old the GPS fix is.

All times are Europe/Lisbon. On failure a tool returns `{"error": "..."}` instead of raising.

## Install

**Quickest** (needs [uv](https://docs.astral.sh/uv/)): no clone needed, it runs straight from PyPI.

```bash
claude mcp add carris-metropolitana -- uvx carris-metropolitana-mcp
```

For Claude Desktop, use `"command": "uvx", "args": ["carris-metropolitana-mcp"]`.

**From source:**

Requires Python 3.10+.

```bash
git clone https://github.com/ThiagoJacobLannes/carris-metropolitana-mcp
cd carris-metropolitana-mcp
python -m pip install -e .
```

On Windows, if you keep code in a cloud-synced folder (Google Drive, OneDrive), create the virtual environment **outside** it: sync clients drop `desktop.ini` files into folders, which can crash packages on startup.

## Add to Claude Code

```bash
claude mcp add carris-metropolitana -- python -m carris_metropolitana_mcp.server
```

## Add to Claude Desktop

Merge `claude_desktop_config.snippet.json` into your Claude Desktop `claude_desktop_config.json`, then restart Claude Desktop:

```json
{
  "mcpServers": {
    "carris-metropolitana": {
      "command": "python",
      "args": ["-m", "carris_metropolitana_mcp.server"]
    }
  }
}
```

## Data source

All data comes from the official Carris Metropolitana open API, `https://api.carrismetropolitana.pt/v2` ([docs](https://docs.carrismetropolitana.pt/), [source](https://github.com/carrismetropolitana/api)). It is public, needs no key, and is the same API the operator's own website and apps use. Carris Metropolitana publishes it as open data; its documentation does not state a specific data licence, so check their terms before redistributing the data itself.

The large, slow-changing lists (all ~12,000 stops, ~700 lines, localities, municipalities) are downloaded once and cached in memory for 12 hours; alerts and vehicle positions are cached for 30 seconds. Set `CARRIS_METROPOLITANA_BASE_URL` to point at a different API base URL.

Real-time estimates only exist for buses that are already running with live tracking; other arrivals are the published timetable. Arrivals cover the current service day only, so late at night the list can be short.

This project is not affiliated with, endorsed by, or connected to Carris Metropolitana or TML.

## License

MIT
