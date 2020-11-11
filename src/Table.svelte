<script lang="ts">
  import {onMount} from 'svelte';
  import type {Heading, ScoutField} from './types';

  export let data: ScoutField[];
  export let headings: Heading[];

  // Support for drag and drop of table rows was heavily inspired by
  // https://htmldom.dev/drag-and-drop-table-row/

  let draggingEle: HTMLTableRowElement;
  let draggingRowIndex: number;
  let editIndex: number;
  let editProperty: string;
  let isDraggingStarted = false;
  let list: HTMLDivElement;
  let placeholder: HTMLDivElement;
  let table: HTMLTableElement;

  // holds current position of mouse relative to dragging element
  let x = 0;
  let y = 0;

  function cloneTable() {
    const rect = table.getBoundingClientRect();
    const width = parseInt(window.getComputedStyle(table).width);

    list = document.createElement('div');
    list.classList.add('clone-list');
    list.style.position = 'absolute';
    list.style.left = `${rect.left}px`;
    list.style.top = `${rect.top}px`;
    table.parentNode.insertBefore(list, table);

    // Hide the original table.
    table.style.visibility = 'hidden';

    table.querySelectorAll('tr').forEach(row => {
      // Create a new table from given row.
      const item = document.createElement('div');
      item.classList.add('draggable');

      const newTable = document.createElement('table');
      newTable.setAttribute('class', 'clone-table');
      newTable.style.width = `${width}px`;

      const newRow = document.createElement('tr');
      const cells = [].slice.call(row.children);
      cells.forEach((cell: HTMLTableCellElement) => {
        const newCell = cell.cloneNode(true) as HTMLTableCellElement;
        newCell.style.width = `${parseInt(
          window.getComputedStyle(cell).width
        )}px`;
        newRow.appendChild(newCell);
      });

      newTable.appendChild(newRow);
      item.appendChild(newTable);
      list.appendChild(item);
    });
  }

  function deleteRow(index: number): void {
    data.splice(index, 1);
    data = data;
  }

  function editCell(index: number, property: string): void {
    editIndex = index;
    editProperty = property;
  }

  function getStyle(heading: Heading) {
    const {width} = heading;
    return width ? `width: ${width}px` : '';
  }

  const handleBlur = (event: FocusEvent) => editProperty = '';

  function handleKey(event: KeyboardEvent): void {
    const {code} = event;
    if (code === 'Enter' || code === 'Escape') editProperty = '';
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

  function isAbove(nodeA: Element, nodeB: Element): boolean {
    // Get the bounding rectangle of nodes.
    const rectA = nodeA.getBoundingClientRect();
    const rectB = nodeB.getBoundingClientRect();

    return rectA.top + rectA.height / 2 < rectB.top + rectB.height / 2;
  }

  function mouseDownHandler(e): void {
    // Get table row containing target.
    let originalRow = e.target.parentNode;
    while (originalRow.nodeName !== 'TR') {
      originalRow = originalRow.parentNode;
    }

    const trs = Array.from(table.querySelectorAll('tr'));
    draggingRowIndex = trs.indexOf(originalRow);

    // Determine mouse position.
    x = e.clientX;
    y = e.clientY;

    // Attach the listeners to document.
    document.addEventListener('mousemove', mouseMoveHandler);
    document.addEventListener('mouseup', mouseUpHandler);
  }

  function mouseMoveHandler(e: MouseEvent): void {
    if (!isDraggingStarted) {
      isDraggingStarted = true;

      cloneTable();

      draggingEle = list.children.item(draggingRowIndex) as HTMLTableRowElement;
      draggingEle.classList.add('dragging');

      // Let placeholder take height of dragging element
      // so next element won't move up.
      placeholder = document.createElement('div');
      placeholder.classList.add('placeholder');
      draggingEle.parentNode.insertBefore(
        placeholder,
        draggingEle.nextSibling
      );
      placeholder.style.height = `${draggingEle.offsetHeight}px`;
    }

    // Set position for dragging element.
    draggingEle.style.position = 'absolute';
    draggingEle.style.top = `${draggingEle.offsetTop + e.clientY - y}px`;
    draggingEle.style.left = `${
      draggingEle.offsetLeft + e.clientX - x
    }px`;

    // Reassign position of mouse.
    x = e.clientX;
    y = e.clientY;

    // The current order is prevEle, draggingEle, placeholder, nextEle.
    const prevEle = draggingEle.previousElementSibling;
    const nextEle = placeholder.nextElementSibling;

    // If the dragging element is above the previous element
    // and the user moves the dragging element to the top,
    // don't allow to drop above the header
    // (which doesn't have `previousElementSibling`).
    if (
      prevEle &&
      prevEle.previousElementSibling &&
      isAbove(draggingEle, prevEle)
    ) {
      // current order -> new order
      // prevEle       -> placeholder
      // draggingEle   -> draggingEle
      // placeholder   -> prevEle
      swapElements(placeholder, draggingEle);
      swapElements(placeholder, prevEle);
      return;
    }

    // If the dragging element is below the next element
    // and the ser moves the dragging element to the bottom ...
    if (nextEle && isAbove(nextEle, draggingEle)) {
      // current order -> new order
      // draggingEle   -> nextEle
      // placeholder   -> placeholder
      // nextEle       -> draggingEle
      swapElements(nextEle, placeholder);
      swapElements(nextEle, draggingEle);
    }
  }

  function mouseUpHandler(): void {
    if (!placeholder) return; // not dragging

    // Remove placeholder.
    if (placeholder) placeholder.parentNode.removeChild(placeholder);

    draggingEle.classList.remove('dragging');
    draggingEle.style.removeProperty('top');
    draggingEle.style.removeProperty('left');
    draggingEle.style.removeProperty('position');

    const endRowIndex = Array.from(list.children).indexOf(draggingEle);

    isDraggingStarted = false;

    // Remove list element.
    list.parentNode.removeChild(list);

    // Move dragged row to endRowIndex.
    let rows = [].slice.call(table.querySelectorAll('tr'));
    draggingRowIndex > endRowIndex
      ? rows[endRowIndex].parentNode.insertBefore(
          rows[draggingRowIndex],
          rows[endRowIndex]
        )
      : rows[endRowIndex].parentNode.insertBefore(
          rows[draggingRowIndex],
          rows[endRowIndex].nextSibling
        );

    // Bring back the table.
    table.style.removeProperty('visibility');

    // Remove handlers of mousemove and mouseup.
    document.removeEventListener('mousemove', mouseMoveHandler);
    document.removeEventListener('mouseup', mouseUpHandler);
  }

  function saveChange(event, index: number, property: string): void {
    data[index][property] = event.target.value;
    data = data;
    if (event.target.nodeName === 'SELECT') editProperty = '';
  }

  const selectAll = (input: HTMLInputElement) => input.select();

  function swapElements(elementA: Element, elementB: Element) {
    const siblingA =
      elementA.nextSibling === elementB ? elementA : elementA.nextSibling;

    // Move elementA to before the elementB.
    elementB.parentNode.insertBefore(elementA, elementB);

    // Move elementB to before the sibling of elementA.
    elementA.parentNode.insertBefore(elementB, siblingA);
  }

  onMount(() => {
    table.querySelectorAll('tr').forEach((row, index: number) => {
      // Ignore the header so the user cannot move it.
      if (index === 0) return;

      // Allow dragging only by first cell of each row.
      const firstCell = row.firstElementChild;
      firstCell.classList.add('draggable');
      firstCell.addEventListener('mousedown', mouseDownHandler);
    });
  });
</script>

<section>
  <table bind:this={table}>
    <thead>
      <tr>
        <th>Drag</th>
        {#each headings as heading}
          <th style={getStyle(heading)}>{heading.title}</th>
        {/each}
        <th>Actions</th>
      </tr>
    </thead>
    <tbody>
      {#each data as obj, index}
        <tr>
          <td class="drag">☰</td>
          {#each headings as heading}
            <td on:dblclick={() => editCell(index, heading.property)}>
              {#if editIndex === index && editProperty === heading.property}
                {#if heading.getOptions}
                  <select
                    on:blur={e => saveChange(e, index, heading.property)}
                    on:change={e => saveChange(e, index, heading.property)}
                    value={obj[heading.property]}>
                    {#each heading.getOptions() as option}
                      <option value={option}>{option}</option>
                    {/each}
                  </select>
                {:else}
                <input
                  on:blur={handleBlur}
                  on:keydown={handleKey}
                  style={getStyle(heading)}
                  type={heading.type}
                  use:selectAll
                  on:change={e => saveChange(e, index, heading.property)}
                  value={data[index][heading.property]}>
                  {/if}
              {:else}<span>{obj[heading.property]}</span>{/if}
            </td>
          {/each}
          <td class="actions">
            <button on:click={() => deleteRow(index)} title="delete">✖🗑</button>
            <button
              on:click={() => insertRowAfter(index)}
              title="insert after">➕</button>
          </td>
        </tr>
      {/each}
    </tbody>
  </table>
</section>

<style>
  .actions button {
    background-color: transparent;
    border: none;
    margin-bottom: 0;
    padding: 0 6px;
  }

  .drag {
    text-align: center;
  }

  input,
  select {
    margin-bottom: 0;
  }

  section :global(.clone-table),
  section :global(table) {
    border-collapse: collapse;
    border: none;
  }

  table :global(.draggable) {
    cursor: move;
    user-select: none;
  }

  section :global(.dragging) {
    background: white;
    z-index: 999;
  }

  section :global(.placeholder) {
    background-color: #edf2f7;
    border: 2px dashed #cbd5e0;
  }

  section :global(td),
  section :global(th) {
    border: solid lightgray 1px;
    padding: 0.5rem;
  }

  section :global(tr) {
    height: 52px
  }
</style>
