<script lang="ts">
  import Avatar from '$lib/components/ui/Avatar.svelte'
  import { getClient } from '$lib/lemmy.js'
  import type { Community, ListingType, Person } from 'lemmy-js-client'
  import { MenuButton, Search } from 'mono-svelte'
  import { createEventDispatcher } from 'svelte'
  import { Icon, XCircle } from 'svelte-hero-icons'
  import { fly } from 'svelte/transition'

  export let q: string = ''

  export let listing_type: ListingType = 'Subscribed'
  export let showWhenEmpty: boolean = false

  const dispatcher = createEventDispatcher<{ select: Person | undefined }>()
</script>

<Search
  search={async (q) => {
    const results = await getClient().search({
      q: q || ' ',
      type_: 'Users',
      limit: 20,
      listing_type: listing_type,
      sort: 'TopAll',
    })

    return results.users.map((c) => c.person)
  }}
  extractName={(c) => `${c.name}@${new URL(c.actor_id).hostname}`}
  on:select
  bind:query={q}
  {showWhenEmpty}
  {...$$restProps}
  let:item
  let:select
>
  <div class="w-full h-full" slot="noresults">
    {#if showWhenEmpty}
      <MenuButton on:click={() => dispatcher('select', undefined)}>
        <Icon src={XCircle} size="16" mini />
        <div class="flex flex-col text-left">
          <span>None</span>
        </div>
      </MenuButton>
    {:else}
      <span class="mx-auto my-auto">No results.</span>
    {/if}
  </div>
  <div in:fly|global={{ y: -4, opacity: 0 }}>
    <MenuButton on:click={() => select(item)}>
      <Avatar url={item.avatar} alt={item.name} width={24} />
      <div class="flex flex-col text-left">
        <span>{item.name}</span>
        <span class="text-xs opacity-80">
          {new URL(item.actor_id).hostname}
        </span>
      </div>
    </MenuButton>
  </div>
</Search>
