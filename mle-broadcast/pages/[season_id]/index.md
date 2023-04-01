<script>
    let seasonId
    $: seasonId = $page.params.season_id
</script>

```week
SELECT sg."start"::date as "start",
       sg."end"::date as "end",
       season.description as season,
       sg.description     as description
FROM schedule_group sg
         INNER JOIN schedule_group_type sgt on sg."typeId" = sgt.id
         INNER JOIN schedule_group season on sg."parentGroupId" = season.id
WHERE sgt.code = 'WEEK'
  AND season.id = {seasonId}
ORDER BY sg.start
```

```fixtures
SELECT away.title as "away", home.title as "home", sf.id as "fixture_id" FROM schedule_fixture sf
    INNER JOIN schedule_group week on sf."scheduleGroupId" = week.id
    INNER JOIN schedule_group season on week."parentGroupId" = season.id
    INNER JOIN franchise_profile home on sf."homeFranchiseId" = home."franchiseId"
    INNER JOIN franchise_profile away on sf."awayFranchiseId" = away."franchiseId"
WHERE season.id = {seasonId}
```

<Tabs>
    {#each week as week}
        <Tab label={week.description}>

        </Tab>
    {/each}
</Tabs>