<script lang="ts">
  import { base } from '$app/paths'
  import { page } from '$app/stores'
  import Header from '$lib/components/extra/Header.svelte'
  import Settings from '$lib/components/extra/Settings.svelte'
  import User from '$lib/components/extra/User.svelte'
  import '$lib/styles/table.scss'
  import '@radar-azdelta/svelte-datatable/style'
  import SettingsImpl from '$lib/helpers/Settings'
  import { userSessionStore as user } from '@radar-azdelta-int/radar-firebase-utils'
  import { createSettings } from '$lib/stores/runes.svelte'

  let { children } = $props()

  let settings = createSettings()

  async function retrieveSettings() {
    const storedSettings = await SettingsImpl.getSettings()
    if (storedSettings) settings.update(storedSettings)
  }

  $effect(() => {
    if ($user?.uid) retrieveSettings()
  })
</script>

<main>
  <header class="header">
    <Header />
    <ul class="page-nav">
      {#if $page.url.pathname !== '/' && $page.url.pathname !== '/Keun'}
        <li><a href="{base}/">File selection</a></li>
      {/if}
    </ul>
    <div class="header-buttons-container" id="settings">
      {#if settings.value}
        <Settings />
        <User />
      {/if}
    </div>
  </header>
  {@render children()}
</main>

<style>
  :global(body) {
    font-family:
      BlinkMacSystemFont,
      -apple-system,
      'Segoe UI',
      'Roboto',
      'Oxygen',
      'Ubuntu',
      'Cantarell',
      'Fira Sans',
      'Droid Sans',
      'Helvetica Neue',
      'Helvetica',
      'Arial',
      sans-serif;
  }

  :global(p),
  :global(input) {
    margin: 0;
    padding: 0;
  }

  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 0 1rem;
  }

  .page-nav {
    list-style: none;
    display: flex;
    align-items: center;
    gap: 2rem;
  }

  a {
    text-decoration: none;
    font-weight: 500;
    color: #454545;
  }

  a:hover,
  a:focus {
    font-weight: 600;
    color: #0070a0;
  }

  .header-buttons-container {
    display: flex;
    gap: 1rem;
    align-items: center;
  }
</style>
