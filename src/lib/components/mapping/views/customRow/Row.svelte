<script lang="ts">
  import { EditableCell } from '@radar-azdelta/svelte-datatable'
  import Config from '$lib/helpers/Config'
  import CustomRow from '$lib/helpers/customRow/CustomRow'
  import Database from '$lib/helpers/Database.svelte'
  import CustomValidation from '$lib/helpers/customRow/CustomValidation'
  import type { ICustomConceptCompact } from '$lib/interfaces/Types'
  import Icon from '$lib/components/extra/Icon.svelte'
  import type { IRowProps } from '$lib/interfaces/Types'
  import { userSessionStore as user } from '@radar-azdelta-int/radar-firebase-utils'
  import { createMappedToConceptIds } from '$lib/stores/runes.svelte'

  let { usagiRow, usagiRowIndex, renderedRow, columns, equivalence, updateError }: IRowProps = $props()

  let mappedToConceptIds = createMappedToConceptIds()
  let row: CustomRow = new CustomRow(renderedRow as ICustomConceptCompact, usagiRow, usagiRowIndex)

  const mapRow = async () => await row.mapCustomConcept('SEMI-APPROVED', equivalence)
  const flagRow = async () => await row.mapCustomConcept('FLAGGED', equivalence)
  const unapproveRow = async () => await row.mapCustomConcept('UNAPPROVED', equivalence)
  const approveRow = async () => await row.mapCustomConcept('APPROVED', equivalence)
  const setRow = async () => (row = new CustomRow(renderedRow as ICustomConceptCompact, usagiRow, usagiRowIndex))

  async function updateCustomConcept(value: string, columnId: string) {
    const editedRow = { ...renderedRow, ...{ [columnId]: value } }
    const result = await CustomValidation.validateRow(editedRow).catch(error => updateError(error))
    if (result) return (renderedRow[columnId] = renderedRow[columnId])
    const { concept_name, concept_class_id, domain_id, vocabulary_id, concept_id, id } = renderedRow
    // TODO: test this further when deleting a custom concepts in the middle of the customs array
    const existingConcept = { concept_name, concept_class_id, domain_id, vocabulary_id, concept_id, id }
    if (columnId === 'concept_name') {
      mappedToConceptIds.value[usagiRow.sourceCode][`custom-${value}`] = mappedToConceptIds.value[usagiRow.sourceCode]?.[`custom-${renderedRow.concept_name}`]
      mappedToConceptIds.update(mappedToConceptIds.value)
    }
    renderedRow[columnId] = value
    const { concept_name: name, concept_class_id: classId, domain_id: domain, vocabulary_id: vocab } = renderedRow
    const newConcept = { concept_name: name, concept_class_id: classId, domain_id: domain, vocabulary_id: vocab, concept_id }
    await Database.updateCustomConcept(newConcept, existingConcept)
  }

  $effect(() => {
    renderedRow
    usagiRow
    usagiRowIndex
    setRow()
  })
</script>

{#if usagiRow}
  {@const mappingStatus = mappedToConceptIds.value[usagiRow.sourceCode]?.[`custom-${renderedRow.concept_name}`]}
  {@const color = Config.colors[mappingStatus ?? 'UNAPPROVED']}
  <td class="actions-cell">
    <div class="actions-grid">
      {#if mappingStatus === 'APPROVED'}
        <button title="Approve mapping" style="background-color: {color}">
          <Icon id="check" width="10px" height="10px" />
        </button>
      {:else if mappingStatus === 'SEMI-APPROVED' && usagiRow.statusSetBy !== $user.name}
        <button onclick={approveRow} title="Approve mapping" style="background-color: {color}">
          <Icon id="check" width="10px" height="10px" />
        </button>
      {:else if mappingStatus === 'SEMI-APPROVED'}
        <button title="Mapped to row" style="background-color: {color}">
          <Icon id="check" width="10px" height="10px" />
        </button>
      {:else}
        <button onclick={mapRow} title="Approve"><Icon id="plus" width="10px" height="10px" /></button>
      {/if}
      {#if mappingStatus === 'FLAGGED'}
        <button title="Flagged mapping" style="background-color: {color}">
          <Icon id="flag" width="10px" height="10px" />
        </button>
      {:else}
        <button onclick={flagRow} title="Flag"><Icon id="flag" width="10px" height="10px" /></button>
      {/if}
      {#if mappingStatus === 'UNAPPROVED'}
        <button title="Unapproved mapping" style="background-color: {color}">
          <Icon id="x" width="10px" height="10px" />
        </button>
      {:else}
        <button onclick={unapproveRow} title="Unapprove"><Icon id="x" width="10px" height="10px" /></button>
      {/if}
    </div>
  </td>
  {#each columns as column, _}
    <td title={renderedRow[column.id].toString()}>
      {#if column.id !== 'concept_id'}
        <EditableCell value={renderedRow[column.id]} changeValue={(value: string) => updateCustomConcept(value, column.id)} />
      {:else}
        <p>{renderedRow[column.id]}</p>
      {/if}
    </td>
  {/each}
{/if}

<style>
  .actions-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(1, 1fr);
    height: max-content;
    max-width: 100%;
  }
</style>
