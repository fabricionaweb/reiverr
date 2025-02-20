<script lang="ts">
	import { TMDB_BACKDROP_SMALL } from '$lib/constants';
	import classNames from 'classnames';
	import { Check } from 'radix-icons-svelte';
	import { fade } from 'svelte/transition';
	import ContextMenu from '../ContextMenu/ContextMenu.svelte';
	import PlayButton from '../PlayButton.svelte';
	import ProgressBar from '../ProgressBar.svelte';
	import ContextMenuItem from '../ContextMenu/ContextMenuItem.svelte';
	import { setJellyfinItemUnwatched, setJellyfinItemWatched } from '$lib/apis/jellyfin/jellyfinApi';
	import { playerState } from '../VideoPlayer/VideoPlayer';
	import { library } from '$lib/stores/library.store';

	export let backdropPath: string;

	export let title = '';
	export let subtitle = '';
	export let episodeNumber: string | undefined = undefined;
	export let runtime = 0;
	export let progress = 0;
	export let watched = false;

	export let jellyfinId: string | undefined = undefined;

	export let size: 'md' | 'dynamic' = 'md';

	function handleSetWatched() {
		if (!jellyfinId) return;

		watched = true;
		setJellyfinItemWatched(jellyfinId).finally(() => library.refreshIn(5000));
	}

	function handleSetUnwatched() {
		if (!jellyfinId) return;

		watched = false;
		setJellyfinItemUnwatched(jellyfinId).finally(() => library.refreshIn(5000));
	}

	function handlePlay() {
		if (!jellyfinId) return;

		playerState.streamJellyfinId(jellyfinId);
	}
</script>

<ContextMenu heading={subtitle}>
	<svelte:fragment slot="menu">
		<ContextMenuItem
			on:click={handleSetWatched}
			disabled={!handleSetWatched || watched || !jellyfinId}
		>
			Mark as watched
		</ContextMenuItem>
		<ContextMenuItem
			on:click={handleSetUnwatched}
			disabled={!handleSetUnwatched || !watched || !jellyfinId}
		>
			Mark as unwatched
		</ContextMenuItem>
	</svelte:fragment>
	<button
		on:click
		class={classNames(
			'aspect-video bg-center bg-cover rounded-lg overflow-hidden transition-opacity shadow-lg selectable flex-shrink-0 placeholder-image relative',
			'flex flex-col px-2 lg:px-3 py-2 gap-2',
			{
				'h-40': size === 'md',
				'h-full': size === 'dynamic',
				group: !!jellyfinId,
				'cursor-default': !jellyfinId
			}
		)}
		style={"background-image: url('" + TMDB_BACKDROP_SMALL + backdropPath + "');"}
		in:fade|global={{ duration: 100, delay: 100 }}
		out:fade|global={{ duration: 100 }}
	>
		<div
			class={classNames(
				'absolute inset-0 transition-opacity group-hover:opacity-0 group-focus-visible:opacity-0',
				{
					'bg-darken': !jellyfinId || watched,
					'bg-gradient-to-t from-darken': !!jellyfinId
				}
			)}
		/>
		<div
			class={classNames(
				'flex-1 flex flex-col justify-between relative group-hover:opacity-0 group-focus-visible:opacity-0 transition-all'
			)}
		>
			<div class="flex justify-between items-center">
				<div>
					<slot name="left-top">
						{#if episodeNumber}
							<p class="text-xs lg:text-sm font-medium text-zinc-300">{episodeNumber}</p>
						{/if}
					</slot>
				</div>
				<div>
					<slot name="right-top">
						{#if runtime}
							<p class="text-xs lg:text-sm font-medium text-zinc-300">
								{runtime} min
							</p>
						{/if}
					</slot>
				</div>
			</div>
			<div class="flex items-end justify-between">
				<slot name="left-bottom">
					<div class="flex flex-col items-start">
						{#if subtitle}
							<h2 class="text-zinc-300 text-sm font-medium">{subtitle}</h2>
						{/if}
						{#if title}
							<h1 class="font-medium text-left line-clamp-2">
								{title}
							</h1>
						{/if}
					</div>
				</slot>
				<slot name="right-bottom">
					{#if watched}
						<div class="flex-shrink-0">
							<Check size={20} class="opacity-80" />
						</div>
					{/if}
				</slot>
			</div>
		</div>
		{#if progress}
			<div class="relative group-hover:opacity-0 transition-opacity">
				<ProgressBar {progress} />
			</div>
		{/if}
		<div class="absolute inset-0 flex items-center justify-center">
			{#if jellyfinId}
				<PlayButton
					on:click={handlePlay}
					class="sm:opacity-0 group-hover:opacity-100 group-focus-visible:opacity-100 transition-opacity"
				/>
			{/if}
		</div>
	</button>
</ContextMenu>
