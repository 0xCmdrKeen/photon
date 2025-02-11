<script lang="ts">
  import Post from '$lib/components/lemmy/post/Post.svelte'
  import Placeholder from '$lib/components/ui/Placeholder.svelte'
  import { userSettings } from '$lib/settings.js'
  import type { PostView } from 'lemmy-js-client'
  import { Badge, Button } from 'mono-svelte'
  import { ArchiveBox, Icon, Minus, Plus } from 'svelte-hero-icons'
  import { expoOut } from 'svelte/easing'
  import { fly, slide } from 'svelte/transition'

  type PostViewWithCrossposts = PostView & {
    withCrossposts: true
    crossposts: PostView[]
  }
  type PostViewWithoutCrossposts = PostView & { withCrossposts?: false }

  export let posts: PostView[]
  export let community: boolean = false

  const addCrosspostProperty = (
    post: PostView,
    crossposts: PostView[]
  ): PostViewWithCrossposts => ({
    ...post,
    crossposts: crossposts,
    withCrossposts: true,
  })

  const combineCrossposts = (
    posts: PostView[]
  ): (PostViewWithCrossposts | PostViewWithoutCrossposts)[] => {
    const urlMap = new Map<
      string,
      PostViewWithCrossposts | PostViewWithoutCrossposts
    >()

    posts.forEach((post) => {
      if (!post.post.url) return

      let existing = urlMap.get(post.post.url)
      if (existing) {
        existing = addCrosspostProperty(existing, [
          ...(existing.withCrossposts ? existing.crossposts : []),
          post,
        ])

        urlMap.set(post.post.url, existing)
      } else {
        urlMap.set(post.post.url, post)
      }
    })

    return Array.from(urlMap.values())
  }

  $: combinedPosts = posts

  let viewPost: number = -1
</script>

<ul
  class="flex flex-col list-none {$userSettings.view == 'card'
    ? 'gap-3 md:gap-4'
    : 'divide-y'} divide-slate-200 dark:divide-zinc-800"
>
  {#if posts.length == 0}
    <div class="h-full grid place-items-center">
      <Placeholder
        icon={ArchiveBox}
        title="No posts"
        description="There are no posts that match this filter."
      >
        <Button href="/communities">
          <Icon src={Plus} size="16" mini slot="prefix" />
          <span>Follow some communities</span>
        </Button>
      </Placeholder>
    </div>
  {:else}
    {#each posts as post, index}
      {#if !($userSettings.hidePosts.deleted && post.post.deleted) && !($userSettings.hidePosts.removed && post.post.removed)}
        <li
          in:fly|global={{
            y: -8,
            duration: 500,
            opacity: 0,
            delay: index < 4 ? index * 100 : 0,
          }}
          class="relative"
        >
          <Post
            hideCommunity={community}
            view={$userSettings.view}
            {post}
            class="transition-all duration-250 {post.withCrossposts &&
            viewPost != post.post.id
              ? ''
              : ''}"
          >
          </Post>
        </li>
      {/if}
    {/each}
  {/if}
</ul>
