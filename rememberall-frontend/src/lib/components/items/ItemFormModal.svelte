<script lang="ts">
	import ItemForm from './ItemForm.svelte';
	import { languageTag, tSync } from '$lib/i18n/index';
	import { createViewportState, calculateModalPosition } from '$lib/utils/viewport.svelte';

	interface Props {
		show?: boolean;
		listId: string;
		isLoading?: boolean;
		onAdd?: (text: string) => void;
	}

	let { show = $bindable(false), listId, isLoading = false, onAdd }: Props = $props();

	let itemFormRef = $state<any>(null);

	let viewportState = createViewportState();
	let modalTopPercent = $derived(calculateModalPosition(viewportState, 300).topPercent);

	function closeModal(): void {
		show = false;
	}

	async function handleDone(): Promise<void> {
		await itemFormRef?.addCurrentText();
		closeModal();
	}
</script>

{#if show}
	<!-- Modal Overlay -->
	<div
		style="background-color: var(--color-overlay-dark);"
		class="fixed inset-0 z-40"
		role="button"
		tabindex="0"
		onclick={closeModal}
		onkeydown={(e) => {
			if (e.key === 'Escape') closeModal();
		}}
		aria-label="Close modal"
	></div>

	<!-- Modal Content - positioned to avoid keyboard -->
	<div
		class="pointer-events-none fixed right-0 left-0 z-50 flex justify-center px-4"
		style="top: {modalTopPercent}%"
	>
		<div
			class="pointer-events-auto w-full max-w-sm space-y-4 rounded-lg border p-4"
			style="background-color: var(--color-bg-secondary); border-color: var(--color-border);"
		>
			<div class="flex items-center justify-between">
				<h2 class="text-xl font-semibold" style="color: var(--color-text-primary);">
					{tSync($languageTag, 'modals.addItem')}
				</h2>
				<button
					type="button"
					onclick={closeModal}
					class="text-2xl leading-none font-bold"
					style="color: var(--color-text-secondary);"
					aria-label="Close modal"
				>
					×
				</button>
			</div>

			<ItemForm bind:this={itemFormRef} {listId} {isLoading} autoFocus={show} {onAdd} />

			<button
				type="button"
				onclick={handleDone}
				disabled={isLoading}
				class="btn btn-secondary w-full"
			>
				{tSync($languageTag, 'modals.done')}
			</button>
		</div>
	</div>
{/if}
