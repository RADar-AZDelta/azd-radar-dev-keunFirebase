<script lang="ts">
  import Row from '$lib/components/mapping/views/customRow/Row.svelte'
  import AddRow from '$lib/components/mapping/views/customRow/AddRow.svelte'
  import type { ICustomRowProps } from '$lib/interfaces/Types'

  let { renderedRow, columns, originalIndex, usagiRow, usagiRowIndex, equivalence, updateError }: ICustomRowProps = $props()
</script>

{#if columns && !originalIndex}
  <AddRow {columns} {originalIndex} {updateError} />
{:else if columns}
  <Row {usagiRow} {usagiRowIndex} {renderedRow} {columns} {equivalence} {updateError} />
{/if}
