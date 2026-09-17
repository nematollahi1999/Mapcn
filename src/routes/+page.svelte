<script lang="ts">
	import type * as MapLibreGL from 'maplibre-gl';
	import { Map, MapArc, MapMarker, MarkerContent, MarkerLabel } from '$lib/components/ui/map';

	type LocationItem = {
		id: string;
		name: string;
		country: string;
		flag: string;
		lng: number;
		lat: number;
		isHub?: boolean;
		limitedBandwidth?: boolean;
	};

	const hub: LocationItem = {
		id: 'london',
		name: 'London',
		country: 'UK',
		flag: '🇬🇧',
		lng: -0.1276,
		lat: 51.5074,
		isHub: true
	};

	const destinations: LocationItem[] = [
		{
			id: 'montreal',
			name: 'Montreal',
			country: 'Canada',
			flag: '🇨🇦',
			lng: -73.5673,
			lat: 45.5017
		},
		{ id: 'texas', name: 'Texas', country: 'USA', flag: '🇺🇸', lng: -96.797, lat: 32.7767 },
		{ id: 'new-york', name: 'New York', country: 'USA', flag: '🇺🇸', lng: -74.006, lat: 40.7128 },
		{
			id: 'los-angeles',
			name: 'Los Angeles',
			country: 'USA',
			flag: '🇺🇸',
			lng: -118.2437,
			lat: 34.0522
		},
		{ id: 'chicago', name: 'Chicago', country: 'USA', flag: '🇺🇸', lng: -87.6298, lat: 41.8781 },
		{ id: 'tampa', name: 'Tampa', country: 'USA', flag: '🇺🇸', lng: -82.4572, lat: 27.9506 },
		{ id: 'miami', name: 'Miami', country: 'USA', flag: '🇺🇸', lng: -80.1918, lat: 25.7617 },
		{
			id: 'frankfurt',
			name: 'Frankfurt',
			country: 'Germany',
			flag: '🇩🇪',
			lng: 8.6821,
			lat: 50.1109
		},
		{ id: 'paris', name: 'Paris', country: 'France', flag: '🇫🇷', lng: 2.3522, lat: 48.8566 },
		{
			id: 'amsterdam',
			name: 'Amsterdam',
			country: 'Netherlands',
			flag: '🇳🇱',
			lng: 4.9041,
			lat: 52.3676
		},
		{ id: 'manchester', name: 'Manchester', country: 'UK', flag: '🇬🇧', lng: -2.2426, lat: 53.4808 },
		{ id: 'vienna', name: 'Vienna', country: 'Austria', flag: '🇦🇹', lng: 16.3738, lat: 48.2082 },
		{ id: 'sofia', name: 'Sofia', country: 'Bulgaria', flag: '🇧🇬', lng: 23.3219, lat: 42.6977 },
		{
			id: 'bucharest',
			name: 'Bucharest',
			country: 'Romania',
			flag: '🇷🇴',
			lng: 26.1025,
			lat: 44.4268
		},
		{ id: 'gdansk', name: 'Gdansk', country: 'Poland', flag: '🇵🇱', lng: 18.6466, lat: 54.352 },
		{
			id: 'singapore',
			name: 'Singapore',
			country: 'Singapore',
			flag: '🇸🇬',
			lng: 103.8198,
			lat: 1.3521,
			limitedBandwidth: true
		},
		{
			id: 'sydney',
			name: 'Sydney',
			country: 'Australia',
			flag: '🇦🇺',
			lng: 151.2093,
			lat: -33.8688,
			limitedBandwidth: true
		},
		{
			id: 'dubai',
			name: 'Dubai',
			country: 'UAE',
			flag: '🇦🇪',
			lng: 55.2708,
			lat: 25.2048,
			limitedBandwidth: true
		},
		{
			id: 'mumbai',
			name: 'Mumbai',
			country: 'India',
			flag: '🇮🇳',
			lng: 72.8777,
			lat: 19.076,
			limitedBandwidth: true
		},
		{ id: 'istanbul', name: 'Istanbul', country: 'Turkey', flag: '🇹🇷', lng: 28.9784, lat: 41.0082 }
	];

	// Display order matching the screenshot layout:
	// Row 1: Montreal, Texas, New York
	// Row 2: Los Angeles, Chicago, Tampa
	// Row 3: Miami, Frankfurt, Paris
	// Row 4: Amsterdam, London (Hub), Manchester
	// Row 5: Vienna, Sofia, Bucharest
	// Row 6: Gdansk, Singapore, Australia Sydney
	// Row 7: Dubai, Mumbai, Istanbul
	const allLocations = [
		destinations[0], // Montreal
		destinations[1], // Texas
		destinations[2], // New York
		destinations[3], // Los Angeles
		destinations[4], // Chicago
		destinations[5], // Tampa
		destinations[6], // Miami
		destinations[7], // Frankfurt
		destinations[8], // Paris
		destinations[9], // Amsterdam
		hub, // London
		destinations[10], // Manchester
		destinations[11], // Vienna
		destinations[12], // Sofia
		destinations[13], // Bucharest
		destinations[14], // Gdansk
		destinations[15], // Singapore
		destinations[16], // Sydney
		destinations[17], // Dubai
		destinations[18], // Mumbai
		destinations[19] // Istanbul
	];

	// Arcs connecting London Hub to all destinations
	const arcs = destinations.map((dest) => ({
		id: `hub-${dest.id}`,
		from: [hub.lng, hub.lat] as [number, number],
		to: [dest.lng, dest.lat] as [number, number]
	}));

	// Target zoom & pitch settings
	const TARGET_ZOOM = 5.5;
	const TARGET_PITCH = 66;

	// Responsive horizontal camera padding: shifts vanishing point so the target sits comfortably further to the right
	function getCameraPadding(): { left: number; top: number; right: number; bottom: number } {
		if (typeof window !== 'undefined' && window.innerWidth >= 768) {
			return { left: 380, top: 0, right: 0, bottom: 0 };
		}
		return { left: 100, top: 0, right: 0, bottom: 0 };
	}

	let map = $state<MapLibreGL.Map | null>(null);
	let selectedLocation = $state<LocationItem>(hub);
	let isPreloaded = $state(false);

	const mapOptions = {
		maxTileCacheSize: 3500,
		maxTileCacheZoomLevels: 12,
		fadeDuration: 0,
		renderWorldCopies: true,
		refreshExpiredTiles: false,
		pitch: TARGET_PITCH,
		attributionControl: false as const,
		padding: getCameraPadding()
	};

	async function prewarmMap() {
		if (!map) return;

		try {
			// 1. Preload 360 globe base coverage
			const globalAngles = [-120, -40, 40, 120];
			for (const lng of globalAngles) {
				map.jumpTo({ center: [lng, 20], zoom: 2 });
				await new Promise((r) => setTimeout(r, 60));
			}

			// 2. Preload city tiles for all targets
			for (const loc of allLocations) {
				map.jumpTo({ center: [loc.lng, loc.lat], zoom: TARGET_ZOOM, padding: getCameraPadding() });
				await new Promise((r) => setTimeout(r, 45));
			}

			// 3. Reset to initial selected location in right 2/3
			map.jumpTo({
				center: [selectedLocation.lng, selectedLocation.lat],
				padding: getCameraPadding(),
				zoom: TARGET_ZOOM,
				pitch: TARGET_PITCH,
				bearing: 20
			});

			// 4. Wait for tile pipeline to idle
			await new Promise<void>((resolve) => {
				const fallback = setTimeout(() => resolve(), 1200);
				if (map?.areTilesLoaded?.()) {
					clearTimeout(fallback);
					resolve();
				} else {
					map?.once('idle', () => {
						clearTimeout(fallback);
						resolve();
					});
				}
			});
		} catch (err) {
			console.warn('Pre-warming notice:', err);
		} finally {
			isPreloaded = true;
		}
	}

	function targetLocation(location: LocationItem) {
		selectedLocation = location;
		if (!map) return;

		map.flyTo({
			center: [location.lng, location.lat],
			padding: getCameraPadding(),
			zoom: TARGET_ZOOM,
			pitch: TARGET_PITCH,
			bearing: (map.getBearing() + 30) % 360,
			speed: 2.0,
			curve: 1.6,
			minZoom: 3.8,
			essential: true
		});
	}
</script>

<div
	class="relative flex min-h-screen w-full flex-col items-center justify-start overflow-x-hidden bg-zinc-950 p-4 font-sans text-zinc-100 sm:p-6 lg:p-10"
>
	<!-- Ambient Background Glow -->
	<div
		class="pointer-events-none absolute inset-0 bg-[radial-gradient(ellipse_70%_50%_at_50%_-10%,rgba(59,130,246,0.18),rgba(255,255,255,0))]"
	></div>

	<!-- ========================================================= -->
	<!-- MAP CONTAINER CARD (ALONE)                                -->
	<!-- ========================================================= -->
	<div
		class="relative h-[420px] w-full max-w-5xl overflow-hidden rounded-3xl border border-white/10 bg-zinc-950 shadow-[0_25px_60px_rgba(0,0,0,0.7)] backdrop-blur-2xl transition-all duration-300 sm:h-[500px]"
	>
		<!-- Preload Indicator -->
		{#if !isPreloaded}
			<div
				class="absolute inset-0 z-40 flex flex-col items-center justify-center bg-zinc-950/95 backdrop-blur-md"
			>
				<div
					class="size-6 animate-spin rounded-full border-2 border-blue-500 border-t-transparent"
				></div>
				<div class="mt-3 text-xs text-zinc-400">Loading globe network...</div>
			</div>
		{/if}

		<!-- Full-width Map Canvas with Smooth Left-to-Right Transparency Gradient -->
		<div
			class="absolute inset-0 h-full w-full overflow-hidden"
			style="-webkit-mask-image: linear-gradient(to right, rgba(0, 0, 0, 0.18) 0%, rgba(0, 0, 0, 0.45) 35%, rgba(0, 0, 0, 0.85) 70%, black 100%); mask-image: linear-gradient(to right, rgba(0, 0, 0, 0.18) 0%, rgba(0, 0, 0, 0.45) 35%, rgba(0, 0, 0, 0.85) 70%, black 100%);"
		>
			<!-- MapLibre Globe Component -->
			<Map
				bind:map
				center={[hub.lng, hub.lat]}
				zoom={TARGET_ZOOM}
				projection={{ type: 'globe' }}
				styles={{
					dark: 'https://basemaps.cartocdn.com/gl/dark-matter-nolabels-gl-style/style.json',
					light: 'https://basemaps.cartocdn.com/gl/positron-nolabels-gl-style/style.json'
				}}
				options={mapOptions}
				onstyleloaded={prewarmMap}
			>
				<MapArc
					data={arcs}
					paint={{
						'line-color': '#38bdf8',
						'line-dasharray': [2, 2],
						'line-width': 2.4
					}}
					interactive={false}
				/>

				<!-- Central Hub Marker (London) -->
				<MapMarker longitude={hub.lng} latitude={hub.lat}>
					<MarkerContent>
						<button
							type="button"
							onclick={() => targetLocation(hub)}
							class="group flex cursor-pointer flex-col items-center"
						>
							<div
								class="size-4 rounded-full border-2 border-white bg-blue-500 shadow-xl shadow-blue-500/60 transition-transform group-hover:scale-125"
							></div>
							<MarkerLabel
								position="top"
								class="rounded-md border border-blue-400/30 bg-zinc-950/90 px-2 py-0.5 text-[11px] font-bold text-blue-300 shadow-lg backdrop-blur"
							>
								{hub.name} (Hub)
							</MarkerLabel>
						</button>
					</MarkerContent>
				</MapMarker>

				<!-- All 20 Destination Markers -->
				{#each destinations as dest (dest.id)}
					<MapMarker longitude={dest.lng} latitude={dest.lat}>
						<MarkerContent>
							<button
								type="button"
								onclick={() => targetLocation(dest)}
								class="group flex cursor-pointer flex-col items-center"
							>
								<div
									class="size-3 rounded-full border border-white bg-emerald-400 shadow-lg transition-transform group-hover:scale-130 {selectedLocation.id ===
									dest.id
										? 'scale-125 ring-4 ring-emerald-400/40'
										: ''}"
								></div>
								<MarkerLabel
									position="top"
									class="rounded-md border border-white/15 bg-zinc-950/85 px-1.5 py-0.5 text-[10px] font-medium text-zinc-200 backdrop-blur"
								>
									{dest.name}
								</MarkerLabel>
							</button>
						</MarkerContent>
					</MapMarker>
				{/each}
			</Map>
		</div>
	</div>

	<!-- ========================================================= -->
	<!-- LOCATION BUTTONS GRID MATCHING SCREENSHOT                 -->
	<!-- ========================================================= -->
	<div class="mt-8 grid w-full max-w-5xl grid-cols-1 gap-3 sm:grid-cols-2 lg:grid-cols-3">
		{#each allLocations as loc (loc.id)}
			{@const isSelected = selectedLocation.id === loc.id}
			<div class="flex flex-col">
				<!-- Optional Limited Bandwidth tag -->
				<div class="h-4">
					{#if loc.limitedBandwidth}
						<span class="text-[10px] font-medium text-zinc-500">Limited Bandwidth</span>
					{/if}
				</div>

				<button
					type="button"
					onclick={() => targetLocation(loc)}
					class="group flex items-center justify-between rounded-xl border p-3.5 text-left transition-all active:scale-[0.98] {isSelected
						? 'border-blue-500/60 bg-blue-500/15 shadow-md ring-1 shadow-blue-500/20 ring-blue-500/30'
						: 'border-white/5 bg-zinc-900/60 hover:border-white/15 hover:bg-zinc-800/60'}"
				>
					<!-- Flag & City Info -->
					<div class="flex items-center gap-3">
						<span class="text-xl leading-none select-none">{loc.flag}</span>
						<div class="flex items-baseline gap-1.5">
							<span
								class="text-sm font-semibold {isSelected
									? 'text-white'
									: 'text-zinc-100 group-hover:text-white'}"
							>
								{loc.name}
							</span>
							<span class="text-xs text-zinc-400">{loc.country}</span>
						</div>
					</div>

					<!-- Radio Selection Indicator Circle -->
					<div
						class="flex size-5 shrink-0 items-center justify-center rounded-full border-2 transition-all {isSelected
							? 'border-blue-500 bg-blue-500'
							: 'border-zinc-700 group-hover:border-zinc-500'}"
					>
						{#if isSelected}
							<div class="size-2 rounded-full bg-white"></div>
						{/if}
					</div>
				</button>
			</div>
		{/each}
	</div>
</div>
