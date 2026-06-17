<script lang="ts">
	import NoteDetails from './details.svelte';
	import MobileDrawer from '@/components/layout/mobile-drawer.svelte';
	import {
		isPageSidebarOpen,
		pageSidebarWidth,
		resizingPageSidebar,
		isNoteDetailSidebarOpen,
		noteDetailSidebarWidth,
		resizingNoteDetailSidebar,
		mobileSidebarOpen,
		mobileDetailsOpen
	} from '@/store';

	// eslint-disable-next-line @typescript-eslint/no-explicit-any
	export let sidebar: any;
</script>

<!-- Desktop layout -->
<div
	class="hidden md:flex flex-col w-full h-[calc(100vh-4.5rem)] bg-secondary-background ml-12 overflow-hidden"
>
	<svelte:component this={sidebar} />
	<div
		class="h-full overflow-y-auto"
		style={`
			margin-left: ${$isPageSidebarOpen ? $pageSidebarWidth : 0}px; 
			margin-right: ${$isNoteDetailSidebarOpen ? $noteDetailSidebarWidth : 0}px; 
			transition: ${
				$resizingPageSidebar || $resizingNoteDetailSidebar
					? 'none'
					: 'margin-left 300ms, margin-right 300ms'
			}
		`}
	>
		<slot />
	</div>
	<NoteDetails />
</div>

<!-- Mobile content: full-width, sidebar + details in drawers -->
<div class="flex md:hidden flex-col w-full min-h-full bg-secondary-background">
	<div class="w-full">
		<slot />
	</div>

	<MobileDrawer open={$mobileSidebarOpen} side="left" title="Files" on:close={() => mobileSidebarOpen.set(false)}>
		<svelte:component this={sidebar} />
	</MobileDrawer>

	<MobileDrawer open={$mobileDetailsOpen} side="bottom" title="Info" on:close={() => mobileDetailsOpen.set(false)}>
		<NoteDetails />
	</MobileDrawer>
</div>
