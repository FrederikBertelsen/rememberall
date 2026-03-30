<script lang="ts">
	import { itemsStore } from '$lib/stores/items.svelte';
	import { languageTag, tSync } from '$lib/i18n/index';
	import type { CreateTodoItemDto, TodoItemDto } from '$lib/api/types';

	interface Props {
		listId: string;
		isLoading?: boolean;
		autoFocus?: boolean;
		onAdd?: (text: string) => void;
	}

	let { listId, isLoading = false, autoFocus = false, onAdd }: Props = $props();

	let text = $state<string>('');
	let error = $state<string | null>(null);
	let selectedSuggestion = $state<TodoItemDto | null>(null);
	let focused = $state(false);
	let inputRef = $state<HTMLInputElement | null>(null);
	let showSuggestions = $state(true);

	// Get all items for this list
	let allItems = $derived(itemsStore.getItemsForList(listId) ?? []);

	// Get all items that match the current text input, sorted by completion count
	let suggestions = $derived(
		text.trim().length === 0
			? []
			: allItems
					.filter((item) => item.text.toLowerCase().includes(text.toLowerCase()))
					.sort((a, b) => b.completionCount - a.completionCount)
					.slice(0, 5) // Limit to 5 suggestions
	);

	// Close suggestions when focus is lost
	$effect(() => {
		if (!focused) {
			showSuggestions = false;
		} else {
			showSuggestions = true;
		}
	});

	// Auto-focus input when form should be focused
	$effect(() => {
		if (autoFocus && inputRef) {
			inputRef.focus();
		}
	});

	function handleSelectSuggestion(item: TodoItemDto): void {
		text = item.text;
		selectedSuggestion = item;
		// Hide suggestions but keep focus so user can click Add
		showSuggestions = false;
		// Keep input focused
		inputRef?.focus();
	}

	function highlightMatch(itemText: string, query: string) {
		const index = itemText.toLowerCase().indexOf(query.toLowerCase());
		if (index === -1) return { before: itemText, match: '', after: '' };

		const before = itemText.slice(0, index);
		const match = itemText.slice(index, index + query.length);
		const after = itemText.slice(index + query.length);

		return { before, match, after };
	}

	async function handleSubmit(e: SubmitEvent): Promise<void> {
		e.preventDefault();
		error = null;

		if (!text.trim()) {
			error = tSync($languageTag, 'form.textRequired');
			return;
		}

		if (text.length > 30) {
			error = tSync($languageTag, 'form.textTooLong');
			return;
		}

		try {
			const trimmedText = text.trim().toLowerCase();

			await performAdd(trimmedText);

			// Clear form but keep modal open
			text = '';
			selectedSuggestion = null;
			showSuggestions = true;
			// Refocus input for next item
			inputRef?.focus();
		} catch (err) {
			error = err instanceof Error ? err.message : tSync($languageTag, 'form.addItemFailed');
		}
	}

	/**
	 * Core add logic extracted so it can be reused by imperative callers.
	 * Does NOT perform empty/length validation — caller is responsible for that.
	 */
	async function performAdd(trimmedText: string): Promise<void> {
		if (onAdd) {
			// Delegate to parent (edit mode) — parent decides what to do
			onAdd(trimmedText);
		} else {
			// If a completed item was selected, mark it incomplete instead of creating new
			if (selectedSuggestion && selectedSuggestion.isCompleted) {
				await itemsStore.incompleteItem(selectedSuggestion.id, listId);
			} else {
				// Check if an item with this exact text already exists
				const existingItem = allItems.find((item) => item.text.toLowerCase() === trimmedText);

				if (existingItem && existingItem.isCompleted) {
					// Reuse the completed item by marking it incomplete
					await itemsStore.incompleteItem(existingItem.id, listId);
				} else if (!existingItem) {
					await itemsStore.createItem({ todoListId: listId, text: trimmedText });
				}
				// If existingItem exists and is not completed, do nothing (already on the list)
			}
		}
	}

	/**
	 * Imperative method for parent components (e.g. modal 'Done') to add current input text.
	 * - Trims current `text`, no-ops if empty after trim.
	 * - Skips max-length validation (per spec) but otherwise uses same add logic.
	 * - Clears input on success.
	 */
	export async function addCurrentText(): Promise<void> {
		error = null;
		const trimmed = text.trim();
		if (!trimmed) return; // no-op when empty after trim

		try {
			const trimmedLower = trimmed.toLowerCase();
			await performAdd(trimmedLower);

			// Clear form but keep modal open
			text = '';
			selectedSuggestion = null;
			showSuggestions = true;
			inputRef?.focus();
		} catch (err) {
			error = err instanceof Error ? err.message : tSync($languageTag, 'form.addItemFailed');
		}
	}
</script>

<form onsubmit={handleSubmit} class="form-group">
	{#if error}
		<div class="alert">
			<p>{error}</p>
		</div>
	{/if}

	<div class="relative">
		<input
			bind:this={inputRef}
			type="text"
			bind:value={text}
			disabled={isLoading}
			placeholder={tSync($languageTag, 'placeholders.itemText')}
			maxlength="30"
			class="form-input w-full"
			autocomplete="off"
			onfocus={() => (focused = true)}
			onblur={() => (focused = false)}
		/>

		{#if focused && showSuggestions && suggestions.length > 0}
			<div
				class="absolute right-0 bottom-full left-0 z-30 mb-2 rounded border"
				style="background-color: var(--color-bg-secondary); border-color: var(--color-border);"
			>
				{#each suggestions as item (item.id)}
					{@const { before, match, after } = highlightMatch(item.text, text)}
					<button
						type="button"
						onmousedown={(e) => {
							e.preventDefault();
							handleSelectSuggestion(item);
						}}
						class="w-full border-b px-3 py-2 text-left text-sm last:border-b-0"
						style="border-color: var(--color-border); color: var(--color-text-primary);"
					>
						{before}<span style="color: white; font-weight: bold;">{match}</span>{after}
					</button>
				{/each}
			</div>
		{/if}
	</div>

	<button type="submit" disabled={isLoading} class="btn btn-primary w-full">
		{isLoading ? tSync($languageTag, 'buttons.adding') : tSync($languageTag, 'buttons.add')}
	</button>

	<div class="flex items-start justify-between">
		<p class="form-hint">{text.length}/30 {tSync($languageTag, 'items.characters')}</p>
		{#if selectedSuggestion}
			<p class="text-xs" style="color: var(--color-accent);">
				{tSync($languageTag, 'items.reuseCompletedItem')}
			</p>
		{/if}
	</div>
</form>
