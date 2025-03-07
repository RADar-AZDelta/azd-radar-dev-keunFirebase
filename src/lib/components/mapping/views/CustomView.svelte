<script lang="ts">
  import DataTable, { type IColumnMetaData, type ITableOptions } from '@radar-azdelta/svelte-datatable'
  import CustomRow from '$lib/components/mapping/views/customRow/CustomRow.svelte'
  import Database, { customConcepts } from '$lib/helpers/Database.svelte'
  import Icon from '$lib/components/extra/Icon.svelte'
  import type { ICustomViewProps, ICustomConceptCompact } from '$lib/interfaces/Types'

  let { selectedRow, selectedRowIndex, equivalence }: ICustomViewProps = $props()

  let data: ICustomConceptCompact[] = $state([])
  const columns: IColumnMetaData[] = [
    {
      id: 'concept_id',
      label: 'Id',
      width: 100,
    },
    {
      id: 'concept_name',
      label: 'name',
    },
    {
      id: 'concept_class_id',
      label: 'className',
    },
    {
      id: 'domain_id',
      label: 'domain',
      width: 175,
    },
    {
      id: 'vocabulary_id',
      label: 'vocabulary',
      width: 100,
    },
  ]

  let errorMessage: string | undefined = $state()
  const options: ITableOptions = {
    actionColumn: true,
    id: 'createCustomConcepts',
    saveOptions: false,
    rowsPerPageOptions: [5, 10, 15],
    rowsPerPage: 15,
  }

  const deleteError = () => (errorMessage = undefined)

  async function updateError(error: string | undefined) {
    errorMessage = error
  }

  async function getAllCustomConcepts() {
    const customConcepts: ICustomConceptCompact[] = (await Database.getCustomConcepts()) as ICustomConceptCompact[]
    const inputRow = { concept_name: '', domain_id: '', vocabulary_id: '', concept_class_id: '', concept_id: -1 }
    data = [inputRow, ...customConcepts]
  }

  async function updateCustoms() {
    const inputRow = { concept_name: '', domain_id: '', vocabulary_id: '', concept_class_id: '', concept_id: -1 }
    data = [inputRow, ...customConcepts]
  }

  $effect(() => {
    getAllCustomConcepts()
  })

  $effect(() => {
    customConcepts
    updateCustoms()
  })
</script>

<div class="custom-concept-container">
  <h2 class="custom-concept-title">Create a custom concept</h2>
  <DataTable {data} {columns} {options}>
    {#snippet rowChild(renderedRow: any, originalIndex: any, index: any, columns: any, option: any)}
      <CustomRow {renderedRow} {columns} {originalIndex} usagiRow={selectedRow} usagiRowIndex={selectedRowIndex} {equivalence} {updateError} />
    {/snippet}
  </DataTable>

  {#if errorMessage}
    <div class="errormessage">
      <p>{errorMessage}</p>
      <button class="errormessage-button" onclick={deleteError}><Icon id="x" /></button>
    </div>
  {/if}
</div>

<style>
  .custom-concept-container {
    padding: 1rem 1rem 3rem 1rem;
  }

  .custom-concept-title {
    font-size: 24px;
    font-weight: 600;
    margin: 0;
    padding: 0 2rem;
    margin-bottom: 2rem;
  }

  .errormessage {
    display: flex;
    width: max-content;
    border-radius: 5px;
    align-items: center;
    gap: 1rem;
    border: 1px solid red;
    background-color: lightcoral;
    padding: 0.5rem 1rem;
  }

  .errormessage-button {
    background-color: inherit;
    border: none;
    cursor: pointer;
    padding: 0;
    height: auto;
  }
</style>
