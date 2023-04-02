---
title: Week 4 FL 3s Hawks @ Knights
---
```away_players
SELECT names.column1 as player_name FROM (VALUES ('Achilles'), ('neas'), ('Villian')) AS names
```
```home_players
SELECT names.column1 as player_name FROM (VALUES ('YenYen'), ('the1337157'), ('Cardinal22')) AS names
```
```fixture
WITH PAGE_VARIABLES AS (
    SELECT 'Match 4' as week,
           'Season 15' as season,
           'FL' as league,
           'Standard' as game_mode,
           'Knights' as playing_team
)
SELECT away.title as "away", home.title as "home", week.description as match, season.description as season, gsgp.code as league, gm.description as game_mode, gsgp."skillGroupId" as leagueid
FROM schedule_fixture
         INNER JOIN franchise_profile away on schedule_fixture."awayFranchiseId" = away."franchiseId"
         INNER JOIN franchise_profile home on schedule_fixture."homeFranchiseId" = home."franchiseId"
         INNER JOIN schedule_group week on schedule_fixture."scheduleGroupId" = week.id
         INNER JOIN schedule_group season on week."parentGroupId" = season.id
         INNER JOIN match_parent mp on schedule_fixture.id = mp."fixtureId"
         INNER JOIN match m on mp.id = m."matchParentId"
         INNER JOIN game_skill_group_profile gsgp on m."skillGroupId" = gsgp."skillGroupId"
         INNER JOIN game_mode gm on m."gameModeId" = gm.id
        INNER JOIN PAGE_VARIABLES vars ON week.description = vars.week
  and season.description = vars.season
  and gsgp.code = vars.league
  and gm.description = vars.game_mode
  and vars.playing_team in (away.title, home.title)
ORDER BY week.start, week.description
```

```win_ntiles
WITH WIN_RATES AS (
    SELECT p.id as playerId,
       COUNT(*) FILTER (WHERE r."homeWon" = psl."isHome") as won,
       COUNT(*) FILTER (WHERE r."homeWon" != psl."isHome") as lost,
       COUNT(*) as played,
       COUNT(*) FILTER (WHERE r."homeWon" = psl."isHome") / COUNT(*)::numeric as rate

       FROM sprocket.player p
    INNER JOIN game_skill_group_profile gsgp on p."skillGroupId" = gsgp."skillGroupId"
    INNER JOIN player_stat_line psl on p.id = psl."playerId"
    INNER JOIN round r on psl."roundId" = r.id
    INNER JOIN match m on r."matchId" = m.id
    INNER JOIN match_parent mp on m."matchParentId" = mp.id
    INNER JOIN schedule_fixture sf on mp."fixtureId" = sf.id
    INNER JOIN schedule_group week on sf."scheduleGroupId" = week.id
    INNER JOIN schedule_group season on week."parentGroupId" = season.id
    INNER JOIN (${fixture}) vars ON season.description = vars.season and gsgp.code = vars.league
GROUP BY p.id, gsgp.id
)
SELECT *,
       ROUND(PERCENT_RANK() over (order by rate)::numeric, 2) * 100 as ntile
       from WIN_RATES
```

```stat_ntiles
WITH AVERAGES AS (SELECT psl."playerId",
                         mp2.name,
                         ROUND(avg((stats -> 'gpi')::numeric), 2)                                        as avg_gpi,
                         ROUND(avg((stats -> 'dpi')::numeric), 2)                                        as avg_dpi,
                         ROUND(avg((stats -> 'opi')::numeric), 2)                                        as avg_opi,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'core' -> 'goals')::numeric), 2) as avg_goals,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'core' -> 'saves')::numeric), 2) as avg_saves,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'core' -> 'shots')::numeric), 2) as avg_shots,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'core' -> 'assists')::numeric),
                               2)                                                                        as avg_assists
                  FROM player_stat_line psl
                           INNER JOIN sprocket.round r on psl."roundId" = r.id
                           INNER JOIN sprocket.match m on r."matchId" = m.id
                           INNER JOIN sprocket.match_parent mp on m."matchParentId" = mp.id
                           INNER JOIN sprocket.schedule_fixture sf on mp."fixtureId" = sf.id
                           INNER JOIN sprocket.player p on psl."playerId" = p.id
                           INNER JOIN sprocket.member_profile mp2 on p."memberId" = mp2."memberId"
                           INNER JOIN (${fixture}) vars ON m."skillGroupId" = vars.leagueId
                  GROUP BY psl."playerId", mp2.name)
SELECT JSON_BUILD_ARRAY(
        json_build_object('stat', 'Sprocket Rating', 'avg', avg_gpi::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_gpi)::numeric, 2) * 100),
        json_build_object('stat', 'Defense Rating', 'avg', avg_dpi::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_dpi)::numeric, 2) * 100),
        json_build_object('stat', 'Offense Rating', 'avg', avg_opi::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_opi)::numeric, 2) * 100),
        json_build_object('stat', 'Goals', 'avg', avg_goals::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_goals)::numeric, 2) * 100),
        json_build_object('stat', 'Saves', 'avg', avg_saves::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_saves)::numeric, 2) * 100),
        json_build_object('stat', 'Shots', 'avg', avg_shots::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_shots)::numeric, 2) * 100),
        json_build_object('stat', 'Assists', 'avg', avg_assists::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_assists)::numeric, 2) * 100)
    ) as stats,
"playerId", name
FROM AVERAGES
```
```misc_ntiles
WITH AVERAGES AS (SELECT psl."playerId",
                         mp2.name,
                         -- Demos
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'demo' -> 'inflicted')::numeric), 2) as avg_demos,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'demo' -> 'taken')::numeric), 2) as avg_demoed,
                         -- Boost
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'boost' -> 'bpm')::numeric), 2) as avg_bpm,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'boost' -> 'amount_stolen')::numeric), 2) as avg_boost_stolen,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'boost' -> 'time_zero_boost')::numeric), 2) as avg_no_boost,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'boost' -> 'count_collected_small')::numeric), 2) as avg_small_boosts,
                         -- Movement
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'movement' -> 'avg_speed')::numeric), 2) as avg_speed,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'movement' -> 'percent_ground')::numeric), 2) as avg_ground_percent,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'movement' -> 'count_powerslide')::numeric), 2) as avg_powerslides,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'movement' -> 'percent_supersonic_speed')::numeric), 2) as avg_supersonic,
                        -- Positioning
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'positioning' -> 'goals_against_while_last_defender')::numeric), 2) as avg_last_man_scored_on,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'positioning' -> 'percent_most_back')::numeric), 2) as avg_last_man,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'positioning' -> 'percent_most_forward')::numeric), 2) as avg_first_man,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'positioning' -> 'percent_infront_ball')::numeric), 2) as avg_infront_ball,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'positioning' -> 'percent_defensive_half')::numeric), 2) as avg_defensive_half,
                         ROUND(avg((stats -> 'otherStats' -> 'stats' -> 'positioning' -> 'percent_offensive_half')::numeric), 2) as avg_offensive_half
                      FROM sprocket.player_stat_line psl
                           INNER JOIN sprocket.round r on psl."roundId" = r.id
                           INNER JOIN sprocket.match m on r."matchId" = m.id
                           INNER JOIN sprocket.match_parent mp on m."matchParentId" = mp.id
                           INNER JOIN sprocket.schedule_fixture sf on mp."fixtureId" = sf.id
                           INNER JOIN sprocket.player p on psl."playerId" = p.id
                           INNER JOIN sprocket.member_profile mp2 on p."memberId" = mp2."memberId"
                           INNER JOIN (${fixture}) vars ON m."skillGroupId" = vars.leagueId
                  GROUP BY psl."playerId", mp2.name)
SELECT JSON_BUILD_ARRAY(
        json_build_object('stat', 'Avg. Demos', 'avg', avg_demos::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_demos)::numeric, 2) * 100),
        json_build_object('stat', 'Avg. Times Demoed', 'avg', avg_demoed::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_demoed)::numeric, 2) * 100),
        json_build_object('stat', 'Avg. BPM', 'avg', avg_bpm::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_bpm)::numeric, 2) * 100),
        json_build_object('stat', 'Avg. Boost Stolen', 'avg', avg_boost_stolen::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_boost_stolen)::numeric, 2) * 100),
        json_build_object('stat', 'Avg. Time w/o boost', 'avg', avg_no_boost::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_no_boost)::numeric, 2) * 100),
        json_build_object('stat', 'Avg. Small Pads', 'avg', avg_small_boosts::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_small_boosts)::numeric, 2) * 100),
        json_build_object('stat', 'Avg. Speed', 'avg', avg_speed::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_speed)::numeric, 2) * 100),
        json_build_object('stat', 'Avg. % of time on ground', 'avg', avg_ground_percent::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_ground_percent)::numeric, 2) * 100),
        json_build_object('stat', 'Avg. Powerslides', 'avg', avg_powerslides::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_powerslides)::numeric, 2) * 100),
        json_build_object('stat', 'Avg. % of time Supersonic', 'avg', avg_supersonic::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_supersonic)::numeric, 2) * 100),
        json_build_object('stat', 'Avg. Times scored on as last man', 'avg', avg_last_man_scored_on::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_last_man_scored_on)::numeric, 2) * 100),
        json_build_object('stat', 'Avg. % of time last man', 'avg', avg_last_man::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_last_man)::numeric, 2) * 100),
        json_build_object('stat', 'Avg. % of time first man', 'avg', avg_first_man::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_first_man)::numeric, 2) * 100),
        json_build_object('stat', 'Avg. % of time in front of ball', 'avg', avg_infront_ball::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_infront_ball)::numeric, 2) * 100),
        json_build_object('stat', 'Avg. % of time behind ball', 'avg', avg_defensive_half::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_defensive_half)::numeric, 2) * 100),
        json_build_object('stat', 'Avg. % of time in offensive half', 'avg', avg_offensive_half::text, 'ntile', ROUND(PERCENT_RANK() OVER (ORDER BY avg_offensive_half)::numeric, 2) * 100)
    ) as stats,
    "playerId",
    name
FROM AVERAGES
```

```away_basic_info
WITH MLE_PLAYER AS (
    SELECT * FROM mledb.player WHERE name in (${away_players})
), SPROCKET_PLAYER AS (
    SELECT player.*, name FROM sprocket.player
            INNER JOIN sprocket.member_profile mp on player."memberId" = mp."memberId"
             WHERE name in (${away_players})
)
SELECT m.name,
       m.team_name as team,
       m.mode_preference,
       s.salary,
       m.id as mleid,
       s.id as sprocketid,
       wins.*
FROM MLE_PLAYER m
    INNER JOIN SPROCKET_PLAYER s ON m.name = s.name
    INNER JOIN ${win_ntiles} wins ON s.id = wins.playerid
    ORDER BY s.salary desc;
```
```away_ntiles
SELECT *
FROM ${stat_ntiles} X
WHERE name IN (${away_players})
```
```away_misc_ntiles
SELECT *
FROM ${misc_ntiles} X
WHERE name IN (${away_players})
```

```home_basic_info
WITH MLE_PLAYER AS (
    SELECT * FROM mledb.player WHERE name in (${home_players})
), SPROCKET_PLAYER AS (
    SELECT player.*, name FROM sprocket.player
            INNER JOIN sprocket.member_profile mp on player."memberId" = mp."memberId"
             WHERE name in (${home_players})
)
SELECT m.name,
       m.team_name as team,
       m.mode_preference,
       s.salary,
       m.id as mleid,
       s.id as sprocketid,
       wins.*
FROM MLE_PLAYER m
    INNER JOIN SPROCKET_PLAYER s ON m.name = s.name
    INNER JOIN ${win_ntiles} wins ON s.id = wins.playerid
    ORDER BY s.salary desc;

```
```home_ntiles
SELECT *
FROM ${stat_ntiles} X
WHERE name IN (${home_players})
```
```home_misc_ntiles
SELECT *
FROM ${misc_ntiles} X
WHERE name IN (${home_players})
```


<!-- 
    Devloper Notes:

        - Query chaining for heavy queries doesn't actually chain queries
          - This means you run heavy queries multiple times
-->

# {fixture[0].away} @ {fixture[0].home}
## {fixture[0].league} {fixture[0].game_mode}

### {fixture[0].away} Roster:
<table>
    <tr>
        <th> Player </th>
        <th> Salary </th>
        <th> Team </th>
        <th> Preferred Mode </th>
        <th> Standing </th>
        <th> Win Rate </th>
    </tr>
{#each away_basic_info as player}
    <tr>
        <td> {player.name} </td>
        <td> {player.salary} </td>
        <td> {player.team} </td>
        <td> {player.mode_preference} </td>
        <td> {player.won} - {player.lost} </td>
        <td> {(player.rate * 100).toFixed(1)}% </td>
    </tr>
{/each}
</table>

### {fixture[0].home} Roster

<table>
    <tr>
        <th> Player </th>
        <th> Salary </th>
        <th> Team </th>
        <th> Preferred Mode </th>
        <th> Standing </th>
        <th> Win Rate </th>
    </tr>
{#each home_basic_info as player}
    <tr>
        <td> {player.name} </td>
        <td> {player.salary} </td>
        <td> {player.team} </td>
        <td> {player.mode_preference} </td>
        <td> {player.won} - {player.lost} </td>
        <td> {(player.rate * 100).toFixed(1)}% </td>
    </tr>
{/each}
</table>

## Core Stat percentiles

### {fixture[0].away}
{#each away_ntiles as player_ntiles}
    <h4>{player_ntiles.name} (Salary {away_basic_info.find(p => p.name === player_ntiles.name).salary})</h4>
    <table>
        <tr>
            <th class="text-left w-full"> Stat </th>
            <th> Avg. Per Game </th>
            <th> %tile in league </th>
        </tr>
    {#each player_ntiles.stats as stat}
        <tr class:good={stat.ntile > 80} class:bad={stat.ntile < 20}>
            <td class="text-left whitespace-nowrap"> {stat.stat} </td>
            <td> {stat.avg} </td>
            <td> Top {stat.ntile}% </td>
        </tr>
    {/each}
        <tr>
            <th class="text-left"> Combined </th>
            <th></th>
            <th> Top {(player_ntiles.stats.reduce((a, v) => a + v.ntile, 0) / player_ntiles.stats.length).toFixed(2)}% </th>
        </tr>
    </table>
{/each}

### {fixture[0].home}
{#each home_ntiles as player_ntiles}
    <h4>{player_ntiles.name} (Salary {home_basic_info.find(p => p.name === player_ntiles.name).salary})</h4>
    <table>
        <tr>
            <th class="text-left"> Stat </th>
            <th> Avg. Per Game </th>
            <th> %tile in league </th>
        </tr>
    {#each player_ntiles.stats as stat}
        <tr class:good={stat.ntile > 80} class:bad={stat.ntile < 20}>
            <td class="text-left whitespace-nowrap"> {stat.stat} </td>
            <td> {stat.avg} </td>
            <td> Top {stat.ntile}% </td>
        </tr>
    {/each}
        <tr>
            <th class="text-left"> Combined </th>
            <th></th>
            <th> Top {(player_ntiles.stats.reduce((a, v) => a + v.ntile, 0) / player_ntiles.stats.length).toFixed(2)}% </th>
        </tr>
    </table>
{/each}

## Misc Stat Percentiles
### {fixture[0].away}
{#each away_misc_ntiles as player_ntiles}
    <h4>{player_ntiles.name} (Salary {away_basic_info.find(p => p.name === player_ntiles.name).salary})</h4>
    <table>
        <tr>
            <th class="text-left"> Stat </th>
            <th> Avg. Per Game </th>
            <th> %tile in league </th>
        </tr>
    {#each player_ntiles.stats as stat}
        <tr class:good={stat.ntile > 80} class:bad={stat.ntile < 20}>
            <td class="text-left"> {stat.stat} </td>
            <td> {stat.avg} </td>
            <td> Top {stat.ntile}% </td>
        </tr>
    {/each}
    </table>
{/each}

### {fixture[0].home}
{#each home_misc_ntiles as player_ntiles}
    <h4>{player_ntiles.name} (Salary {home_basic_info.find(p => p.name === player_ntiles.name).salary})</h4>
    <table>
        <tr>
            <th class="text-left"> Stat </th>
            <th> Avg. Per Game </th>
            <th> %tile in league </th>
        </tr>
    {#each player_ntiles.stats as stat}
        <tr class:good={stat.ntile > 80} class:bad={stat.ntile < 20}>
            <td class="text-left"> {stat.stat} </td>
            <td> {stat.avg} </td>
            <td> Top {stat.ntile}% </td>
        </tr>
    {/each}
    </table>
{/each}

<style lang="postcss"> 
    td, th { @apply max-w-sm min-w-fit whitespace-nowrap; }

    tr {
        &.good td { @apply bg-green-200; }
        &.bad td { @apply bg-red-200; }
    }
</style>