<script lang="ts">
  import type {Heading, ScoutField} from './types';

  export let caption: string;
  export let data: ScoutField[];
  export let headings: Heading[];

  let editIndex;
  let editProperty;

  function deleteRow(index: number): void {
    data.splice(index, 1);
    data = data;
  }

  function editCell(index: number, property: string): void {
    editIndex = index;
    editProperty = property;
  }

  function handleKey(event): void {
    const {code} = event;
    if (code === 'Enter' || code === 'Escape' || code === 'Tab') {
      editProperty = '';
    }
  }

  function insertRowAfter(index: number): void {
    const item: ScoutField = {
      scoutName: '',
      growerName: '',
      fieldName: '',
      acres: 0
    };
    data.splice(index + 1, 0, item);
    data = data;
  }

  const moveDown = (index: number): void => swap(index, index + 1);
  const moveUp = (index: number): void => swap(index, index - 1);
  function swap(i1: number, i2: number): void {
    const temp = data[i1];
    data[i1] = data[i2];
    data[i2] = temp;
    data = data;
  }

  const selectAll = node => node.select();
</script>

<table>
  {#if caption}
    <caption>{caption}</caption>
  {/if}
  <thead>
    <tr>
      {#each headings as heading}
        <th>{heading.title}</th>
      {/each}
      <th>Actions</th>
    </tr>
  </thead>
  <tbody>
    {#each data as obj, index}
      <tr>
        {#each headings as heading}
          <td on:dblclick={() => editCell(index, heading.property)}>
            {#if editIndex === index && editProperty === heading.property}
              <input
                on:keydown={handleKey}
                use:selectAll
                bind:value={data[index][heading.property]} />
            {:else}<span>{obj[heading.property]}</span>{/if}
          </td>
        {/each}
        <td class="actions">
          <button on:click={() => deleteRow(index)} title="delete">✖</button>
          {#if index > 0}
            <button on:click={() => moveUp(index)} title="move up">▲</button>
          {/if}
          {#if index < data.length - 1}
            <button
              on:click={() => moveDown(index)}
              title="move down">▼</button>
          {/if}
          <button
            on:click={() => insertRowAfter(index)}
            title="insert after">➕</button>
        </td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .actions button {
    background-color: transparent;
    border: none;
  }

  caption {
    font-size: 1.5rem;
    font-weight: bold;
    margin-bottom: 0.5rem;
  }

  table {
    border-collapse: collapse;
  }

  td,
  th {
    border: solid lightgray 1px;
    padding: 0.5rem;
  }
</style>
