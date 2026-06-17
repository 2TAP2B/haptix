<script lang="ts">
	import migrations from '$lib/database/migrations/migrations.sql?raw';
	import seed from '$lib/database/migrations/seed.sql?raw';
	import { loadSettings } from '@/api/settings';
import Footer from '@/components/layout/footer.svelte';
import Header from '@/components/layout/header.svelte';
import Sidebar from '@/components/layout/sidebar.svelte';
import MobileNav from '@/components/layout/mobile-nav.svelte';
import Command from '@/components/shared/command-menu/command.svelte';
import Icon from '@/components/shared/icon.svelte';
import { db, pgClient } from '@/database/client';
import { collection as collectionTable } from '@/database/schema';
import {
	collection,
	isMobile,
	mobileSidebarOpen,
	mobileView,
	activeFile
} from '@/store';
	import { createDeviceDetector, createMobileStore } from '@/utils';
	import '@haptic/ui/app.web.css';
	import { ModeWatcher } from 'mode-watcher';
	import { onMount } from 'svelte';

	// Device detector
	const device = createDeviceDetector();
	const mobile = createMobileStore();

	// Migrate database
	async function migrateDatabase() {
		try {
			await pgClient.exec(migrations);

			// Seed database
			await pgClient.exec(seed);
		} catch (error) {
			console.log('Table already exists');
		}
	}

	// Load latest collection
	async function loadLatestCollection() {
		const collections = await db.select().from(collectionTable);

		if (!collections || collections.length === 0) return;

		// Get collection with latest lastOpened date
		const latestCollection = collections.reduce((prev, current) =>
			prev.lastOpened > current.lastOpened ? prev : current
		);

		collection.set(latestCollection.path);
	}

	onMount(async () => {
		// Migrate database
		await migrateDatabase();

		console.log(await db.select().from(collectionTable));
		// Load latest collection on mount
		await loadLatestCollection();

		// Load app & collection settings
		loadSettings(true, true);
	});

	// Sync isMobile store with media query
	$: isMobile.set($mobile);

	// Auto-switch to editor view when a file is selected on mobile
	$: if ($isMobile && $activeFile) {
		mobileView.set('editor');
	} else if ($isMobile && !$activeFile) {
		mobileView.set('files');
	}
</script>

<svelte:head>
	<title>Haptic</title>
	<meta
		name="description"
		content="Haptic is a new local-first & privacy-focused home for your markdown notes. It's a minimalistic, lightweight and fast note-taking app that's designed to be distraction-free."
	/>
	<meta
		name="keywords"
		content="Haptic, Note-taking, Markdown, Local-first, Privacy-focused, Open-source, Online Markdown Editor, Fast Note-taking, Minimalistic Design"
	/>
	<meta name="author" content="Haptic" />
	<meta name="robots" content="index, follow" />
	<meta name="viewport" content="width=device-width, initial-scale=1" />
	<meta name="theme-color" content="#0F0F0F" />
	<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />

	<!-- Open Graph -->
	<meta property="og:site_name" content="Haptic" />
	<meta property="og:locale" content="en" />
	<meta property="og:type" content="website" />
	<meta property="og:url" content="https://haptic.md/" />
	<meta property="og:title" content="Haptic - Write Notes at the speed of touch" />
	<meta
		property="og:description"
		content="Haptic is a new local-first & privacy-focused home for your markdown notes. It's a minimalistic, lightweight and fast note-taking app that's designed to be distraction-free."
	/>
	<meta property="og:image" content="https://haptic.md/landing.png" />
	<meta property="og:image:alt" content="Haptic - Markdown Editor" />
	<meta property="og:image:width" content="1200" />
	<meta property="og:image:height" content="627" />

	<!-- Twitter -->
	<meta property="twitter:card" content="summary_large_image" />
	<meta property="twitter:url" content="https://haptic.md/" />
	<meta property="twitter:title" content="Haptic - Write Notes at the speed of touch" />
	<meta
		property="twitter:description"
		content="Haptic is a new local-first & privacy-focused home for your markdown notes. It's a minimalistic, lightweight and fast note-taking app that's designed to be distraction-free."
	/>
	<meta property="twitter:image" content="https://haptic.md/landing.png" />

	{#if import.meta.env.PROD}
		<script
			defer
			src="https://cloud.umami.is/script.js"
			data-website-id="279d8c15-20ea-4cc9-91b0-647c90767f15"
		></script>
		<script async src="https://cdn.seline.so/seline.js" data-token="d028e058129b859"></script>
	{/if}
</svelte:head>

<Command />
<ModeWatcher />

<!-- Desktop layout -->
<div class="hidden md:flex md:flex-col md:w-full">
	<Header />
	<Sidebar />
	<main class="flex min-h-screen w-full items-center justify-center">
		<slot />
	</main>
	<Footer />
</div>

<!-- Mobile layout -->
<div class="flex md:hidden flex-col w-full min-h-[100dvh]">
	<!-- Mobile header -->
	<div
		class="fixed top-0 left-0 right-0 h-12 border-b bg-background z-40 flex items-center justify-between px-3"
	>
		<button
			class="h-8 w-8 flex items-center justify-center rounded-md hover:bg-accent fill-muted-foreground hover:fill-foreground transition-all"
			on:click={() => {
				if ($activeFile && $mobileView === 'editor') {
					mobileView.set('files');
				} else {
					mobileSidebarOpen.set(true);
				}
			}}
		>
			{#if $activeFile && $mobileView === 'editor'}
				<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
					<path d="M19 12H5" />
					<polyline points="12 19 5 12 12 5" />
				</svg>
			{:else}
				<svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
					<line x1="3" y1="6" x2="21" y2="6" />
					<line x1="3" y1="12" x2="21" y2="12" />
					<line x1="3" y1="18" x2="21" y2="18" />
				</svg>
			{/if}
		</button>
		<p class="text-sm font-medium text-foreground/85 truncate mx-2">
			{$collection?.split('/').pop() || 'Haptic'}
		</p>
		<div class="flex items-center gap-1">
			<button
				class="h-8 w-8 flex items-center justify-center rounded-md hover:bg-accent fill-muted-foreground hover:fill-foreground transition-all"
				on:click={() => {
					document.dispatchEvent(new KeyboardEvent('keydown', { key: 'o', metaKey: true }));
				}}
			>
				<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
					<path d="M22 19a2 2 0 0 1-2 2H4a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h5l2 3h9a2 2 0 0 1 2 2z" />
				</svg>
			</button>
			<button
				class="h-8 w-8 flex items-center justify-center rounded-md hover:bg-accent fill-muted-foreground hover:fill-foreground transition-all"
				on:click={() => {
					document.dispatchEvent(new KeyboardEvent('keydown', { key: ',', metaKey: true }));
				}}
			>
				<svg width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
					<circle cx="12" cy="12" r="3" />
					<path d="M19.4 15a1.65 1.65 0 0 0 .33 1.82l.06.06a2 2 0 0 1 0 2.83 2 2 0 0 1-2.83 0l-.06-.06a1.65 1.65 0 0 0-1.82-.33 1.65 1.65 0 0 0-1 1.51V21a2 2 0 0 1-2 2 2 2 0 0 1-2-2v-.09A1.65 1.65 0 0 0 9 19.4a1.65 1.65 0 0 0-1.82.33l-.06.06a2 2 0 0 1-2.83 0 2 2 0 0 1 0-2.83l.06-.06A1.65 1.65 0 0 0 4.68 15a1.65 1.65 0 0 0-1.51-1H3a2 2 0 0 1-2-2 2 2 0 0 1 2-2h.09A1.65 1.65 0 0 0 4.6 9a1.65 1.65 0 0 0-.33-1.82l-.06-.06a2 2 0 0 1 0-2.83 2 2 0 0 1 2.83 0l.06.06A1.65 1.65 0 0 0 9 4.68a1.65 1.65 0 0 0 1-1.51V3a2 2 0 0 1 2-2 2 2 0 0 1 2 2v.09a1.65 1.65 0 0 0 1 1.51 1.65 1.65 0 0 0 1.82-.33l.06-.06a2 2 0 0 1 2.83 0 2 2 0 0 1 0 2.83l-.06.06A1.65 1.65 0 0 0 19.4 9a1.65 1.65 0 0 0 1.51 1H21a2 2 0 0 1 2 2 2 2 0 0 1-2 2h-.09a1.65 1.65 0 0 0-1.51 1z" />
				</svg>
			</button>
		</div>
	</div>

	<!-- Mobile content area -->
	<main
		class="flex-1 pt-12 pb-14 w-full overflow-y-auto"
	>
		<slot />
	</main>

	<MobileNav />
</div>

<style>
	/* Custom scrollbar */
	:global(::-webkit-scrollbar) {
		width: 14px;
	}

	:global(::-webkit-scrollbar-thumb) {
		border: 4px solid rgba(0, 0, 0, 0);
		background-clip: padding-box;
		border-radius: 50px;
		background-color: hsl(var(--border) / 1);

		&:hover {
			background-color: hsl(var(--foreground) / 0.15);
		}
	}
</style>
