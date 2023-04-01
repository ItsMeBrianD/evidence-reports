---
title: Week 4 FL 3s Hawks @ Knights
---
```stat_ntiles
WITH SKILL_GROUP AS (SELECT *
                     FROM game_skill_group_profile
                     WHERE code = 'FL'),
     AVERAGES AS (SELECT psl."playerId",
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
                           INNER JOIN round r on psl."roundId" = r.id
                           INNER JOIN match m on r."matchId" = m.id
                           INNER JOIN match_parent mp on m."matchParentId" = mp.id
                           INNER JOIN schedule_fixture sf on mp."fixtureId" = sf.id
                           INNER JOIN SKILL_GROUP gsgp on m."skillGroupId" = gsgp."skillGroupId"
                           INNER JOIN player p on psl."playerId" = p.id
                           INNER JOIN member_profile mp2 on p."memberId" = mp2."memberId"
                  GROUP BY psl."playerId", mp2.name)
SELECT JSON_BUILD_ARRAY(
        json_build_object('stat', 'Sprocket Rating', 'avg', avg_gpi::text, 'ntile', ntile(100) OVER (order by avg_gpi)),
        json_build_object('stat', 'Defense Rating', 'avg', avg_dpi::text, 'ntile', ntile(100) OVER (order by avg_dpi)),
        json_build_object('stat', 'Offense Rating', 'avg', avg_opi::text, 'ntile', ntile(100) OVER (order by avg_opi)),
        json_build_object('stat', 'Goals', 'avg', avg_goals::text, 'ntile', ntile(100) OVER (order by avg_goals)),
        json_build_object('stat', 'Saves', 'avg', avg_saves::text, 'ntile', ntile(100) OVER (order by avg_saves)),
        json_build_object('stat', 'Shots', 'avg', avg_shots::text, 'ntile', ntile(100) OVER (order by avg_shots)),
        json_build_object('stat', 'Assists', 'avg', avg_assists::text, 'ntile', ntile(100) OVER (order by avg_assists))
    ) as stats,
"playerId", name
FROM AVERAGES
```
```misc_ntiles
WITH SKILL_GROUP AS (SELECT *
                     FROM game_skill_group_profile
                     WHERE code = 'FL'),
     AVERAGES AS (SELECT psl."playerId",
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
                      FROM player_stat_line psl
                           INNER JOIN round r on psl."roundId" = r.id
                           INNER JOIN match m on r."matchId" = m.id
                           INNER JOIN match_parent mp on m."matchParentId" = mp.id
                           INNER JOIN schedule_fixture sf on mp."fixtureId" = sf.id
                           INNER JOIN SKILL_GROUP gsgp on m."skillGroupId" = gsgp."skillGroupId"
                           INNER JOIN player p on psl."playerId" = p.id
                           INNER JOIN member_profile mp2 on p."memberId" = mp2."memberId"
                  GROUP BY psl."playerId", mp2.name)
SELECT JSON_BUILD_ARRAY(
        json_build_object('stat', 'Avg. Demos', 'avg', avg_demos::text, 'ntile', ntile(100) over (order by avg_demos)),
        json_build_object('stat', 'Avg. Times Demoed', 'avg', avg_demoed::text, 'ntile', ntile(100) over (order by avg_demoed)),
        json_build_object('stat', 'Avg. BPM', 'avg', avg_bpm::text, 'ntile', ntile(100) over (order by avg_bpm)),
        json_build_object('stat', 'Avg. Boost Stolen', 'avg', avg_boost_stolen::text, 'ntile', ntile(100) over (order by avg_boost_stolen)),
        json_build_object('stat', 'Avg. Time w/o boost', 'avg', avg_no_boost::text, 'ntile', ntile(100) over (order by avg_no_boost)),
        json_build_object('stat', 'Avg. Small Pads', 'avg', avg_small_boosts::text, 'ntile', ntile(100) over (order by avg_small_boosts)),
        json_build_object('stat', 'Avg. Speed', 'avg', avg_speed::text, 'ntile', ntile(100) over (order by avg_speed)),
        json_build_object('stat', 'Avg. % of time on ground', 'avg', avg_ground_percent::text, 'ntile', ntile(100) over (order by avg_ground_percent)),
        json_build_object('stat', 'Avg. Powerslides', 'avg', avg_powerslides::text, 'ntile', ntile(100) over (order by avg_powerslides)),
        json_build_object('stat', 'Avg. % of time Supersonic', 'avg', avg_supersonic::text, 'ntile', ntile(100) over (order by avg_supersonic)),
        json_build_object('stat', 'Avg. Times scored on as last man', 'avg', avg_last_man_scored_on::text, 'ntile', ntile(100) over (order by avg_last_man_scored_on)),
        json_build_object('stat', 'Avg. % of time last man', 'avg', avg_last_man::text, 'ntile', ntile(100) over (order by avg_last_man)),
        json_build_object('stat', 'Avg. % of time first man', 'avg', avg_first_man::text, 'ntile', ntile(100) over (order by avg_first_man)),
        json_build_object('stat', 'Avg. % of time in front of ball', 'avg', avg_infront_ball::text, 'ntile', ntile(100) over (order by avg_infront_ball)),
        json_build_object('stat', 'Avg. % of time behind ball', 'avg', avg_defensive_half::text, 'ntile', ntile(100) over (order by avg_defensive_half)),
        json_build_object('stat', 'Avg. % of time in offensive half', 'avg', avg_offensive_half::text, 'ntile', ntile(100) over (order by avg_offensive_half))
    ) as stats,
    "playerId",
    name
FROM AVERAGES
```


```hawks_players
SELECT names.column1 as player_name FROM (VALUES ('Achilles'), ('neas'), ('Villian')) AS names
```
```hawks_basic_info
WITH MLE_PLAYER AS (
    SELECT * FROM mledb.player WHERE name in (${hawks_players})
), SPROCKET_PLAYER AS (
    SELECT player.*, name FROM player
            INNER JOIN member_profile mp on player."memberId" = mp."memberId"
             WHERE name in (${hawks_players})
)
SELECT m.name, m.team_name as team, m.mode_preference, s.salary, m.id as mleid, s.id as sprocketid FROM MLE_PLAYER m
    INNER JOIN SPROCKET_PLAYER s ON m.name = s.name;
```
```hawks_ntiles
SELECT *
FROM ${stat_ntiles} X
WHERE name IN (${hawks_players})
```
```hawks_misc_ntiles
SELECT *
FROM ${misc_ntiles} X
WHERE name IN (${hawks_players})
```

```knights_players
SELECT names.column1 as player_name FROM (VALUES ('YenYen'), ('the1337157'), ('Cardinal22')) AS names
```
```knights_basic_info
WITH MLE_PLAYER AS (
    SELECT * FROM mledb.player WHERE name in (${knights_players})
), SPROCKET_PLAYER AS (
    SELECT player.*, name FROM player
            INNER JOIN member_profile mp on player."memberId" = mp."memberId"
             WHERE name in (${knights_players})
)
SELECT m.name, m.team_name as team, m.mode_preference, s.salary, m.id as mleid, s.id as sprocketid FROM MLE_PLAYER m
    INNER JOIN SPROCKET_PLAYER s ON m.name = s.name;
```
```knights_ntiles
SELECT *
FROM ${stat_ntiles} X
WHERE name IN (${knights_players})
```
```knights_misc_ntiles
SELECT *
FROM ${misc_ntiles} X
WHERE name IN (${knights_players})
```

# Hawks @ Knights
## FL Standard

### Hawks Roster:
<table>
    <tr>
        <th> Player </th>
        <th> Salary </th>
        <th> Team </th>
        <th> Preferred Mode </th>
    </tr>
{#each hawks_basic_info as player}
    <tr>
        <td> {player.name} </td>
        <td> {player.salary} </td>
        <td> {player.team} </td>
        <td> {player.mode_preference} </td>
    </tr>
{/each}
</table>

### Knights Roster

<table>
    <tr>
        <th> Player </th>
        <th> Salary </th>
        <th> Team </th>
        <th> Preferred Mode </th>
    </tr>
{#each knights_basic_info as player}
    <tr>
        <td> {player.name} </td>
        <td> {player.salary} </td>
        <td> {player.team} </td>
        <td> {player.mode_preference} </td>
    </tr>
{/each}
</table>

## Core Stat percentiles

### Hawks
{#each hawks_ntiles as player_ntiles}
    <h4>{player_ntiles.name} (Salary {hawks_basic_info.find(p => p.name === player_ntiles.name).salary})</h4>
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

### Knights
{#each knights_ntiles as player_ntiles}
    <h4>{player_ntiles.name} (Salary {knights_basic_info.find(p => p.name === player_ntiles.name).salary})</h4>
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
### Hawks
{#each hawks_misc_ntiles as player_ntiles}
    <h4>{player_ntiles.name} (Salary {hawks_basic_info.find(p => p.name === player_ntiles.name).salary})</h4>
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

### Knights
{#each knights_misc_ntiles as player_ntiles}
    <h4>{player_ntiles.name} (Salary {knights_basic_info.find(p => p.name === player_ntiles.name).salary})</h4>
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