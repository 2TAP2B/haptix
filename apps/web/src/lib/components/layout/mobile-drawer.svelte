<script lang="ts">
	import { cn } from '@haptic/ui/lib/utils';
	import { createEventDispatcher } from 'svelte';
	import { fly } from 'svelte/transition';

	export let open: boolean = false;
	export let side: 'left' | 'right' | 'bottom' = 'left';
	export let title: string = '';

	const dispatch = createEventDispatcher();

	function close() {
		dispatch('close');
	}

	function handleBackdropClick(e: MouseEvent) {
		if (e.target === e.currentTarget) {
			close();
		}
	}
</script>

<svelte:window on:keydown={(e) => { if (e.key === 'Escape' && open) close(); }} />

{#if open}
	<div
		class="fixed inset-0 bg-black/40 z-40 md:hidden"
		on:click={handleBackdropClick}
		role="presentation"
	>
		{#if side === 'bottom'}
			<div
				class={cn(
					'fixed bottom-0 left-0 right-0 max-h-[85vh] bg-background rounded-t-xl overflow-y-auto z-50',
					'shadow-xl'
				)}
				transition:fly={{ y: 200, duration: 250 }}
			>
				<div
					class="sticky top-0 bg-background flex items-center justify-between px-4 py-3 border-b z-10"
				>
					<h2 class="text-sm font-medium">{title}</h2>
					<button
						class="h-7 w-7 flex items-center justify-center rounded-md hover:bg-accent fill-muted-foreground hover:fill-foreground transition-all"
						on:click={close}
					>
						<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
							<line x1="18" y1="6" x2="6" y2="18" />
							<line x1="6" y1="6" x2="18" y2="18" />
						</svg>
					</button>
				</div>
				<div class="p-0">
					<slot />
				</div>
			</div>
		{:else}
			<div
				class={cn(
					'fixed top-0 bottom-0 w-[85vw] max-w-[320px] bg-background z-50 overflow-y-auto shadow-xl',
					side === 'left' ? 'left-0' : 'right-0'
				)}
				transition:fly={{ x: side === 'left' ? -320 : 320, duration: 250 }}
			>
				<div
					class="sticky top-0 bg-background flex items-center justify-between px-4 py-3 border-b z-10"
				>
					<h2 class="text-sm font-medium">{title}</h2>
					<button
						class="h-7 w-7 flex items-center justify-center rounded-md hover:bg-accent fill-muted-foreground hover:fill-foreground transition-all"
						on:click={close}
					>
						<svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
							<line x1="18" y1="6" x2="6" y2="18" />
							<line x1="6" y1="6" x2="18" y2="18" />
						</svg>
					</button>
				</div>
				<div class="p-0">
					<slot />
				</div>
			</div>
		{/if}
	</div>
{/if}
