<script lang="ts">
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';
	import { mobileDetailsOpen, mobileView, activeFile } from '@/store';
	import Icon from '$lib/components/shared/icon.svelte';
	import { cn } from '@haptic/ui/lib/utils';

	$: path = $page.url.pathname;

	function isActive(section: string) {
		return path === `/${section}` || path.startsWith(`/${section}/`);
	}

	function navigateTo(section: string) {
		const target = `/${section}`;
		if (path !== target) {
			goto(target);
		}
	}
</script>

<nav
	class="fixed bottom-0 left-0 right-0 h-14 border-t bg-background z-50 flex items-center justify-around px-2 safe-area-bottom md:hidden"
>
	<button
		class={cn(
			'flex flex-col items-center justify-center gap-0.5 h-full flex-1 rounded-lg transition-all',
			isActive('notes') ? 'text-foreground' : 'text-muted-foreground'
		)}
		on:click={() => navigateTo('notes')}
	>
		<Icon
			name="inboxFull"
			class={cn(
				'w-5 h-5 transition-all',
				isActive('notes') ? 'fill-foreground' : 'fill-muted-foreground'
			)}
		/>
		<span class="text-[10px] font-medium">Notes</span>
	</button>

	<button
		class={cn(
			'flex flex-col items-center justify-center gap-0.5 h-full flex-1 rounded-lg transition-all',
			isActive('daily') ? 'text-foreground' : 'text-muted-foreground'
		)}
		on:click={() => navigateTo('daily')}
	>
		<Icon
			name="calendarEdit"
			class={cn(
				'w-5 h-5 transition-all',
				isActive('daily') ? 'fill-foreground' : 'fill-muted-foreground'
			)}
		/>
		<span class="text-[10px] font-medium">Daily</span>
	</button>

	<button
		class={cn(
			'flex flex-col items-center justify-center gap-0.5 h-full flex-1 rounded-lg transition-all',
			isActive('tasks') ? 'text-foreground' : 'text-muted-foreground'
		)}
		on:click={() => navigateTo('tasks')}
	>
		<Icon
			name="checkSquare"
			class={cn(
				'w-5 h-5 transition-all',
				isActive('tasks') ? 'fill-foreground' : 'fill-muted-foreground'
			)}
		/>
		<span class="text-[10px] font-medium">Tasks</span>
	</button>

	<button
		class={cn(
			'flex flex-col items-center justify-center gap-0.5 h-full flex-1 rounded-lg transition-all',
			$activeFile && $mobileView === 'editor' ? 'text-foreground' : 'text-muted-foreground'
		)}
		on:click={() => {
			if ($activeFile) {
				mobileView.set($mobileView === 'editor' ? 'files' : 'editor');
			}
		}}
	>
		<Icon
			name="editPencil"
			class={cn(
				'w-5 h-5 transition-all',
				$activeFile && $mobileView === 'editor' ? 'fill-foreground' : 'fill-muted-foreground'
			)}
		/>
		<span class="text-[10px] font-medium">Editor</span>
	</button>

	<button
		class={cn(
			'flex flex-col items-center justify-center gap-0.5 h-full flex-1 rounded-lg transition-all',
			$mobileDetailsOpen ? 'text-foreground' : 'text-muted-foreground'
		)}
		on:click={() => {
			mobileDetailsOpen.update((v) => !v);
		}}
	>
		<Icon
			name="identityGhost"
			class={cn(
				'w-5 h-5 transition-all',
				$mobileDetailsOpen ? 'fill-foreground' : 'fill-muted-foreground'
			)}
		/>
		<span class="text-[10px] font-medium">Info</span>
	</button>
</nav>

<style>
	.safe-area-bottom {
		padding-bottom: env(safe-area-inset-bottom, 0px);
	}
</style>
