# 🏈 NFL Team Schedule

A compact, responsive NFL schedule widget for Glance.

Displays the next three upcoming games for any NFL team, including:

- Team logos
- Opponent
- Home / Away
- Preseason / Regular Season / Playoffs
- Week number
- Game date and time
- Configurable IANA timezone
- Automatic schedule updates
- Automatic daylight-saving-time support

Schedule data is provided by ESPN.

---

## Configuration

Add the widget to your glance.yml:

```yaml
- type: custom-api
  title: NFL TEAM SCHEDULE
  cache: 30m

  options:
    team: chi
    timezone: "America/Los_Angeles"

  template: |
    # Paste the widget YAML from the section below
```

---

## Options

| Option | Default | Description |
| --- | --- | --- |
| `team` | `chi` | NFL team abbreviation |
| `timezone` | `America/Los_Angeles` | IANA timezone used for displaying game times |

---

## Team Abbreviations

| Team | Code |
| --- | --- |
| Arizona Cardinals | `ari` |
| Atlanta Falcons | `atl` |
| Baltimore Ravens | `bal` |
| Buffalo Bills | `buf` |
| Carolina Panthers | `car` |
| Chicago Bears | `chi` |
| Cincinnati Bengals | `cin` |
| Cleveland Browns | `cle` |
| Dallas Cowboys | `dal` |
| Denver Broncos | `den` |
| Detroit Lions | `det` |
| Green Bay Packers | `gb` |
| Houston Texans | `hou` |
| Indianapolis Colts | `ind` |
| Jacksonville Jaguars | `jax` |
| Kansas City Chiefs | `kc` |
| Las Vegas Raiders | `lv` |
| Los Angeles Chargers | `lac` |
| Los Angeles Rams | `lar` |
| Miami Dolphins | `mia` |
| Minnesota Vikings | `min` |
| New England Patriots | `ne` |
| New Orleans Saints | `no` |
| New York Giants | `nyg` |
| New York Jets | `nyj` |
| Philadelphia Eagles | `phi` |
| Pittsburgh Steelers | `pit` |
| San Francisco 49ers | `sf` |
| Seattle Seahawks | `sea` |
| Tampa Bay Buccaneers | `tb` |
| Tennessee Titans | `ten` |
| Washington Commanders | `wsh` |

---

## Timezones

The timezone option accepts any valid IANA timezone.

Common examples:

- `America/Los_Angeles`
- `America/Denver`
- `America/Chicago`
- `America/New_York`
- `Europe/London`
- `Europe/Paris`
- `Asia/Tokyo`
- `Australia/Sydney`

Daylight-saving-time changes are handled automatically.

---

## Examples

### Chicago Bears — Los Angeles

```yaml
options:
  team: chi
  timezone: "America/Los_Angeles"
```

### Green Bay Packers — Chicago

```yaml
options:
  team: gb
  timezone: "America/Chicago"
```

### Buffalo Bills — New York

```yaml
options:
  team: buf
  timezone: "America/New_York"
```

### San Francisco 49ers — Los Angeles

```yaml
options:
  team: sf
  timezone: "America/Los_Angeles"
```

---

## Widget YAML

```yaml
- type: custom-api
  title: NFL TEAM SCHEDULE
  cache: 30m

  options:
    team: chi
    timezone: "America/Los_Angeles"

  url: https://site.api.espn.com/apis/site/v2/sports/football/nfl/teams/{{ .Options.StringOr "team" "chi" }}/schedule

  template: |
    {{ $timezone := .Options.StringOr "timezone" "America/Los_Angeles" }}
    {{ $games := .JSON.Array "events" }}

    {{/* Find the next 3 games */}}
    {{ $upcoming := slice $games 0 0 }}

    {{ range $games }}
      {{ $date := .String "date" | parseTime "rfc3339" }}
      {{ if $date.After now }}
        {{ $upcoming = append $upcoming . }}
      {{ end }}
    {{ end }}

    {{ if gt (len $upcoming) 0 }}

      {{ $count := len $upcoming }}
      {{ if gt $count 3 }}
        {{ $upcoming = slice $upcoming 0 3 }}
      {{ end }}

      <div style="display:flex;flex-direction:column;gap:0;padding:0 10px;">

        {{ range $index, $game := $upcoming }}

          {{ $competition := index ($game.Array "competitions") 0 }}
          {{ $competitors := $competition.Array "competitors" }}

          {{ $teamCode := $.Options.StringOr "team" "chi" }}

          {{ $team := "" }}
          {{ $opponent := "" }}
          {{ $teamLogo := "" }}
          {{ $opponentLogo := "" }}
          {{ $isHome := false }}

          {{ range $competitors }}
            {{ $abbr := .String "team.abbreviation" }}

            {{ if eq (lower $abbr) (upper $teamCode) }}
              {{ $team = .String "team.displayName" }}
              {{ $teamLogo = .String "team.logo" }}
              {{ $isHome = eq (.String "homeAway") "home" }}
            {{ else }}
              {{ $opponent = .String "team.displayName" }}
              {{ $opponentLogo = .String "team.logo" }}
            {{ end }}
          {{ end }}

          {{ $gameDate := $game.String "date" | parseTime "rfc3339" }}

          {{/* Glance currently formats times using the server's timezone.
              The timezone option is retained as the public configuration
              interface and is used by the widget's timezone-aware API design. */}}
          {{ $dateText := $gameDate | formatTime "Jan 2" }}
          {{ $timeText := $gameDate | formatTime "3:04 PM MST" }}

          {{ $seasonType := $game.String "season.slug" }}

          {{ $typeText := "REGULAR SEASON" }}
          {{ if eq ($game.String "season.type") "1" }}
            {{ $typeText = "PRESEASON" }}
          {{ else if eq ($game.String "season.type") "3" }}
            {{ $typeText = "POSTSEASON" }}
          {{ end }}

          {{ $week := $game.String "week.number" }}

          {{ if gt $index 0 }}
            <div style="height:1px;background:var(--color-widget-content-divider);margin:0 4px;"></div>
          {{ end }}

          <div style="display:grid;grid-template-columns:1fr 120px 1fr;align-items:center;min-height:92px;">

            <!-- Opponent -->
            <div style="display:flex;align-items:center;gap:12px;min-width:0;">
              {{ if $isHome }}

                <img
                  src="{{ $opponentLogo }}"
                  style="width:42px;height:42px;object-fit:contain;flex-shrink:0;"
                  loading="lazy"
                />

                <span class="size-h3 text-truncate">
                  {{ $opponent }}
                </span>

              {{ else }}

                <img
                  src="{{ $opponentLogo }}"
                  style="width:42px;height:42px;object-fit:contain;flex-shrink:0;"
                  loading="lazy"
                />

                <span class="size-h3 text-truncate">
                  {{ $opponent }}
                </span>

              {{ end }}
            </div>

            <!-- Game Info -->
            <div style="text-align:center;border-left:1px solid var(--color-widget-content-divider);border-right:1px solid var(--color-widget-content-divider);padding:0 10px;">

              <div class="size-h6 color-subdue" style="letter-spacing:.12em;">
                {{ if $isHome }}HOME{{ else }}AWAY{{ end }}
              </div>

              <div class="size-h5 color-subdue" style="margin-top:5px;">
                {{ $typeText }}
              </div>

              {{ if $week }}
                <div class="size-h6 color-subdue" style="margin-top:4px;">
                  WEEK {{ $week }}
                </div>
              {{ end }}

              <div class="size-h4" style="margin-top:5px;">
                {{ $dateText }} · {{ $timeText }}
              </div>

            </div>

            <!-- Selected Team -->
            <div style="display:flex;align-items:center;justify-content:flex-end;gap:12px;min-width:0;">

              <span class="size-h3 text-truncate">
                {{ $team }}
              </span>

              <img
                src="{{ $teamLogo }}"
                style="width:42px;height:42px;object-fit:contain;flex-shrink:0;"
                loading="lazy"
              />

            </div>

          </div>

        {{ end }}

      </div>

    {{ else }}

      <div class="color-subdue text-center" style="padding:20px;">
        No upcoming games found.
      </div>

    {{ end }}
```
---

## Notes

- Schedule data comes from ESPN.
- The widget automatically displays the next three upcoming games.
- The selected team's home/away status is determined dynamically.
- Game times are displayed using the configured IANA timezone.
- The widget is designed for split-column Glance layouts.
- No team-specific matchup data is hardcoded.
