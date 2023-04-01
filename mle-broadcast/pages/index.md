# MLE Broadcast Stats Helper

This report is designed to look up matches, streams, and some player stats to give
casters additional context & content going into streams.


```seasons
SELECT sg.id as season_id,
       sg."start"::date,
       sg."end"::date,
       sg.description as season
FROM schedule_group sg
         INNER JOIN schedule_group_type sgt on sg."typeId" = sgt.id
WHERE sgt.code = 'SEASON';
```

## View season stats
<ul>
{#each seasons as season}
    <li><a href="/{season.season_id}">{season.season}</a></li>
{/each}
</ul>