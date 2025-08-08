<script lang="ts">
	import Fuse from 'fuse.js';
	import Bolt from '$lib/components/icons/Bolt.svelte';
	import { onMount, getContext } from 'svelte';
	import { settings, NetsafeAI } from '$lib/stores';
	import { WEBUI_VERSION } from '$lib/constants';

	const i18n = getContext('i18n');

	export let suggestionPrompts = [];
	export let className = '';
	export let inputValue = '';
	export let onSelect = (e) => {};

	let sortedPrompts = [];

	const fuseOptions = {
		keys: ['content', 'title'],
		threshold: 0.5
	};

	let fuse;
	let filteredPrompts = [];

	// Initialize Fuse
	$: fuse = new Fuse(sortedPrompts, fuseOptions);

	// Update the filteredPrompts if inputValue changes
	// Only increase version if something wirklich geändert hat
	$: getFilteredPrompts(inputValue);

	// Helper function to check if arrays are the same
	// (based on unique IDs oder content)
	function arraysEqual(a, b) {
		if (a.length !== b.length) return false;
		for (let i = 0; i < a.length; i++) {
			if ((a[i].id ?? a[i].content) !== (b[i].id ?? b[i].content)) {
				return false;
			}
		}
		return true;
	}

	const getFilteredPrompts = (inputValue) => {
		if (inputValue.length > 500) {
			filteredPrompts = [];
		} else {
			const newFilteredPrompts =
				inputValue.trim() && fuse
					? fuse.search(inputValue.trim()).map((result) => result.item)
					: sortedPrompts;

			// Compare with the oldFilteredPrompts
			// If there's a difference, update array + version
			if (!arraysEqual(filteredPrompts, newFilteredPrompts)) {
				filteredPrompts = newFilteredPrompts;
			}
		}
	};

	$: if (suggestionPrompts) {
		sortedPrompts = [...(suggestionPrompts ?? [])].sort(() => Math.random() - 0.5);
		getFilteredPrompts(inputValue);
	}
</script>

		<div class="mb-1 flex gap-1 text-xs font-medium items-center text-gray-600 dark:text-gray-400">
			{#if filteredPrompts.length > 0}
				<Bolt />
				{$i18n.t('Suggested')}
			{:else}

		<!-- Keine Vorschläge -->
		<div style="max-width: 640px; margin: 2.5rem auto; border-radius: 1.5rem; box-shadow: 0 2px 24px 0 #ececf1; background: #ffffff; display: flex; overflow: hidden;"> 
			<div style="flex: 1; text-align: center; padding: 2.2rem 2rem; display: flex; flex-direction: column; justify-content: center;">
				<div style="font-weight: 600; font-size: 1.22rem; margin-bottom: 0.9rem; color: #36373c; letter-spacing: 0.02em;">
					Datenschutz
				</div>
				<div style="font-size: 1.04rem; color: #292b32; line-height: 1.55;">
					Keine Weitergabe, keine Überwachung durch Dritte.<br>
					Kontrolle über alle Daten.
				</div>
			</div>
			<!-- Middle-Line-Netsafe -->
			<div style=" width:1px; background: linear-gradient(to bottom, #ededf4 10%, #d5d7de 50%, #ededf4 90%); margin: 1.2rem 0; border-radius: 1rem;">
			</div>
			<div style="flex: 1; text-align: center; padding: 2.2rem 2rem; display: flex; flex-direction: column; justify-content: center;">
				<div style="font-weight: 600; font-size: 1.22rem; margin-bottom: 0.9rem; color: #36373c; letter-spacing: 0.02em;">
					Swiss Hosting
				</div>
				<div style="font-size: 1.04rem; color: #292b32; line-height: 1.55;">
					Hosting ausschliesslich in der Schweiz und physische Sicherheit Ihrer Daten.
				</div>
			</div>
		</div>

		<!-- <div
			class="flex w-full {$settings?.landingPageMode === 'chat'
				? ' -mt-1'
				: 'text-center items-center justify-center'}  self-start text-gray-600 dark:text-gray-400"
		>
			NetsafeAI
		</div> -->
	{/if}
</div>

<div class="h-40 w-full">
	{#if filteredPrompts.length > 0}
		<div role="list" class="max-h-40 overflow-auto scrollbar-none items-start {className}">
			{#each filteredPrompts as prompt, idx (prompt.id || prompt.content)}
				<!-- svelte-ignore a11y-no-interactive-element-to-noninteractive-role -->
				<button
					role="listitem"
					class="waterfall flex flex-col flex-1 shrink-0 w-full justify-between
				       px-3 py-2 rounded-xl bg-transparent hover:bg-black/5
				       dark:hover:bg-white/5 transition group"
					style="animation-delay: {idx * 60}ms"
					on:click={() => onSelect({ type: 'prompt', data: prompt.content })}
				>
					<div class="flex flex-col text-left">
						{#if prompt.title && prompt.title[0] !== ''}
							<div
								class="font-medium dark:text-gray-300 dark:group-hover:text-gray-200 transition line-clamp-1"
							>
								{prompt.title[0]}
							</div>
							<div class="text-xs text-gray-600 dark:text-gray-400 font-normal line-clamp-1">
								{prompt.title[1]}
							</div>
						{:else}
							<div
								class="font-medium dark:text-gray-300 dark:group-hover:text-gray-200 transition line-clamp-1"
							>
								{prompt.content}
							</div>
							<div class="text-xs text-gray-600 dark:text-gray-400 font-normal line-clamp-1">
								{$i18n.t('Prompt')}
							</div>
						{/if}
					</div>
				</button>
			{/each}
		</div>
	{/if}
</div>

<style>
	/* Waterfall animation for the suggestions */
	@keyframes fadeInUp {
		0% {
			opacity: 0;
			transform: translateY(20px);
		}
		100% {
			opacity: 1;
			transform: translateY(0);
		}
	}

	.waterfall {
		opacity: 0;
		animation-name: fadeInUp;
		animation-duration: 200ms;
		animation-fill-mode: forwards;
		animation-timing-function: ease;
	}
</style>
