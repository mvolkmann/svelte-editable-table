<script lang="ts">
  import Table from './Table.svelte';
  import type {Heading, ScoutField} from './types';

  const headings: Heading[] = [
    {title: 'Scout', property: 'scoutName'},
    {title: 'Grower', property: 'growerName'},
    {title: 'Field', property: 'fieldName'},
    {title: 'Acres', property: 'acres'}
  ];

  const data: ScoutField[] = [
    {scoutName: 's1', growerName: 'g1', fieldName: 'f1', acres: 100},
    {scoutName: 's1', growerName: 'g2', fieldName: 'f2', acres: 200},
    {scoutName: 's2', growerName: 'g3', fieldName: 'f3', acres: 300},
    {scoutName: 's2', growerName: 'g4', fieldName: 'f4', acres: 400}
  ];
</script>

<main>
  <Table caption="Next 7 Days" {headings} {data} />
</main>

<style>
</style>
