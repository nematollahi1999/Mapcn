<div align="center">

# 🗺️ Mapcn

### Beautiful, declarative map components for **Svelte 5** and **Tailwind CSS**.

_Crafted with the **shadcn** philosophy, powered by **MapLibre GL**._

[![Svelte 5](https://img.shields.io/badge/Svelte-5.x-orange.svg?style=flat-square&logo=svelte)](https://svelte.dev/)
[![MapLibre GL](https://img.shields.io/badge/MapLibre_GL-6.x-blue.svg?style=flat-square&logo=maplibre)](https://maplibre.org/)
[![Tailwind CSS v4](https://img.shields.io/badge/Tailwind_CSS-v4.x-38bdf8.svg?style=flat-square&logo=tailwindcss)](https://tailwindcss.com/)
[![TypeScript](https://img.shields.io/badge/TypeScript-Ready-3178c6.svg?style=flat-square&logo=typescript)](https://www.typescriptlang.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](LICENSE)

<br />

<p align="center">
  <b>Mapcn</b> brings the composable, copy-paste architecture of <code>shadcn/ui</code> to maps in Svelte 5.<br />
  Build high-performance 2D maps and 3D globes with rich interactive HTML markers, curved network arcs, animated routes, GeoJSON layers, clustering, and dark-mode basemaps with zero boilerplate.
</p>

</div>

---

## ✨ Features

- **⚡ Svelte 5 Runes Native** — Built from scratch with `$state`, `$derived`, `$effect`, and modern Svelte 5 snippets.
- **🌍 3D Globe & 2D Projections** — Seamlessly toggle between flat Mercator and interactive 3D globes with tilt and pitch.
- **📍 Rich Declarative Markers** — Full HTML/SVG freedom inside marker snippets, supporting animated pulses, labels, popups, and hover tooltips.
- **🌐 Curved Network Arcs (`MapArc`)** — Render great-circle & quadratic Bezier curves connecting coordinates with customizable curvature and hover states.
- **🛣️ Route Paths & Progress (`MapRoute`)** — Draw lines, visualize animated transit progress (`RouteProgress`), and attach waypoints/vehicles (`RouteMarker`).
- **🫧 Point Clustering (`MapClusterLayer`)** — Automatically cluster thousands of points with smooth expansion and custom color thresholds.
- **🗺️ GeoJSON Layers (`MapGeoJSON`)** — Render vector geometries, polygons, and boundaries with reactive fill/line styling and click handling.
- **🌗 Automatic Theme Switching** — Out-of-the-box light and dark basemaps that track your application's theme (`.dark` class or `data-theme`).
- **🎛️ Tailored Map Controls (`MapControls`)** — Minimalist, floating zoom, 3D compass with pitch tracking, locate, and fullscreen buttons.
- **🧩 Shadcn Philosophy** — Own your code. No black-box dependencies—copy, paste, and customize freely to match your design system.

---

## 📦 Installation

### 1. Install Dependencies

In your SvelteKit / Svelte 5 project:

```bash
pnpm add maplibre-gl @lucide/svelte cn
pnpm add -D @types/geojson
```

> **Note:** Mapcn requires **Svelte 5** and works seamlessly with **Tailwind CSS v4** or **v3**.

### 2. Copy the Map Component Directory

Copy `src/lib/components/ui/map` directly into your project:

```
src/lib/components/ui/map/
├── Map.svelte             # Core map container & lifecycle
├── MapArc.svelte          # Great-circle curved arcs
├── MapClusterLayer.svelte # GeoJSON point clustering
├── MapControls.svelte     # Floating navigation controls
├── MapGeoJSON.svelte      # Polygons & line layers
├── MapMarker.svelte       # Marker coordinate wrapper
├── MarkerContent.svelte   # Marker HTML/SVG content
├── MarkerLabel.svelte     # Marker badge / text label
├── MarkerPopup.svelte     # Click/hover popup attached to marker
├── MarkerTooltip.svelte   # Hover tooltip
├── MapPopup.svelte        # Standalone coordinate popup
├── MapRoute.svelte        # Route path layer
├── RouteProgress.svelte   # Animated route progress
├── RouteMarker.svelte     # Moving waypoint on a route
├── route.ts               # Geometry & distance math
├── theme.ts               # Basemap theme resolver
├── use-map.svelte.ts      # Reactive MapLibre context hook
└── index.ts               # Barrel exports
```

---

## 🚀 Quick Start

### Basic Interactive Map

```svelte
<script lang="ts">
	import { Map, MapControls, MapMarker, MarkerContent, MarkerLabel } from '$lib/components/ui/map';
</script>

<div
	class="h-[500px] w-full overflow-hidden rounded-2xl border border-zinc-200 dark:border-zinc-800"
>
	<Map
		center={[-0.1276, 51.5074]}
		zoom={12}
		styles={{
			dark: 'https://basemaps.cartocdn.com/gl/dark-matter-nolabels-gl-style/style.json',
			light: 'https://basemaps.cartocdn.com/gl/positron-nolabels-gl-style/style.json'
		}}
	>
		<!-- Map Navigation Controls -->
		<MapControls position="bottom-right" showZoom showCompass showLocate />

		<!-- Custom Marker -->
		<MapMarker longitude={-0.1276} latitude={51.5074}>
			<MarkerContent>
				<div
					class="size-4 animate-pulse rounded-full border-2 border-white bg-blue-500 shadow-lg"
				/>
				<MarkerLabel
					position="top"
					class="rounded bg-zinc-900/90 px-2 py-0.5 text-xs text-white shadow"
				>
					London
				</MarkerLabel>
			</MarkerContent>
		</MapMarker>
	</Map>
</div>
```

---

## 💡 Examples & Recipes

### 1. 3D Globe with Network Arcs

Render a global flight or server distribution network:

```svelte
<script lang="ts">
	import { Map, MapArc, MapMarker, MarkerContent, MarkerLabel } from '$lib/components/ui/map';

	const hub = { name: 'London', lng: -0.1276, lat: 51.5074 };
	const destinations = [
		{ name: 'New York', lng: -74.006, lat: 40.7128 },
		{ name: 'Tokyo', lng: 139.6917, lat: 35.6895 },
		{ name: 'Sydney', lng: 151.2093, lat: -33.8688 }
	];

	const arcs = destinations.map((dest) => ({
		id: `arc-${dest.name}`,
		from: [hub.lng, hub.lat] as [number, number],
		to: [dest.lng, dest.lat] as [number, number]
	}));
</script>

<div class="h-[600px] w-full overflow-hidden rounded-3xl border border-white/10 bg-zinc-950">
	<Map
		center={[hub.lng, hub.lat]}
		zoom={2.5}
		projection={{ type: 'globe' }}
		options={{ pitch: 45 }}
	>
		<!-- Curved great-circle lines -->
		<MapArc
			data={arcs}
			curvature={0.25}
			paint={{
				'line-color': '#38bdf8',
				'line-dasharray': [2, 2],
				'line-width': 2
			}}
		/>

		<!-- Hub Marker -->
		<MapMarker longitude={hub.lng} latitude={hub.lat}>
			<MarkerContent>
				<div class="size-3.5 rounded-full border-2 border-white bg-blue-500 shadow-md" />
				<MarkerLabel position="top">{hub.name} (Hub)</MarkerLabel>
			</MarkerContent>
		</MapMarker>

		<!-- Destination Markers -->
		{#each destinations as dest (dest.name)}
			<MapMarker longitude={dest.lng} latitude={dest.lat}>
				<MarkerContent>
					<div class="size-2.5 rounded-full border border-white bg-emerald-400" />
					<MarkerLabel position="top">{dest.name}</MarkerLabel>
				</MarkerContent>
			</MapMarker>
		{/each}
	</Map>
</div>
```

---

### 2. Markers with Popups & Tooltips

```svelte
<script lang="ts">
	import {
		Map,
		MapMarker,
		MarkerContent,
		MarkerPopup,
		MarkerTooltip
	} from '$lib/components/ui/map';
</script>

<Map center={[-74.006, 40.7128]} zoom={13}>
	<MapMarker longitude={-74.006} latitude={40.7128}>
		<MarkerContent>
			<div
				class="flex size-5 items-center justify-center rounded-full bg-indigo-600 text-xs text-white"
			>
				📍
			</div>
		</MarkerContent>

		<!-- Hover tooltip -->
		<MarkerTooltip>Click for details</MarkerTooltip>

		<!-- Interactive Popup on click -->
		<MarkerPopup closeButton class="p-3">
			<h4 class="text-sm font-bold">New York City</h4>
			<p class="mt-1 text-xs text-zinc-500">Status: Operational</p>
		</MarkerPopup>
	</MapMarker>
</Map>
```

---

### 3. Routes & Animated Progress Tracking

```svelte
<script lang="ts">
	import { Map, MapRoute, RouteProgress, RouteMarker } from '$lib/components/ui/map';

	const routeCoordinates: [number, number][] = [
		[-0.1419, 51.5014], // Buckingham Palace
		[-0.1276, 51.5074], // Trafalgar Square
		[-0.1195, 51.5033] // London Eye
	];

	let progress = $state(0.4); // 40% completed
</script>

<Map center={[-0.1276, 51.5074]} zoom={14}>
	<MapRoute coordinates={routeCoordinates} color="#94a3b8" width={4} {progress}>
		<!-- Traveled path overlay with active color -->
		<RouteProgress color="#3b82f6" width={4} />

		<!-- Current position marker moving along the route -->
		<RouteMarker>
			<div
				class="flex size-6 items-center justify-center rounded-full border-2 border-white bg-blue-600 text-[10px] text-white shadow"
			>
				🚗
			</div>
		</RouteMarker>
	</MapRoute>
</Map>
```

---

### 4. High-Density Point Clustering

Effortlessly handle hundreds or thousands of points:

```svelte
<script lang="ts">
	import { Map, MapClusterLayer } from '$lib/components/ui/map';

	// Pass either a GeoJSON FeatureCollection object or an API URL
	const earthquakesUrl = 'https://maplibre.org/maplibre-gl-js/test/data/earthquakes.geojson';
</script>

<Map center={[-103.59, 40.71]} zoom={3}>
	<MapClusterLayer
		data={earthquakesUrl}
		clusterRadius={50}
		clusterMaxZoom={14}
		clusterColors={['#60a5fa', '#3b82f6', '#1d4ed8']}
		clusterThresholds={[50, 250]}
		onpointclick={(feature, coords) => console.log('Point:', feature)}
	/>
</Map>
```

---

### 5. GeoJSON Polygons & Custom Layers

```svelte
<script lang="ts">
	import { Map, MapGeoJSON } from '$lib/components/ui/map';

	const zoneGeoJSON = {
		type: 'FeatureCollection',
		features: [
			{
				type: 'Feature',
				geometry: {
					type: 'Polygon',
					coordinates: [/* ... */]
				},
				properties: { name: 'Delivery Zone A' }
			}
		]
	};
</script>

<Map center={[-74.006, 40.7128]} zoom={12}>
	<MapGeoJSON
		data={zoneGeoJSON}
		fillPaint={{
			'fill-color': '#3b82f6',
			'fill-opacity': 0.25
		}}
		linePaint={{
			'line-color': '#1d4ed8',
			'line-width': 2
		}}
		fillHoverPaint={{
			'fill-color': '#60a5fa',
			'fill-opacity': 0.45
		}}
		interactive
		onclick={(e) => alert(e.feature.properties.name)}
	/>
</Map>
```

---

### 6. Programmatic Camera & `useMap()` Hook

Access the underlying `maplibre-gl.Map` instance anywhere:

```svelte
<!-- Inside parent component -->
<script lang="ts">
	import type * as MapLibreGL from 'maplibre-gl';
	import { Map } from '$lib/components/ui/map';

	let mapInstance = $state<MapLibreGL.Map | null>(null);

	function flyToCity(lng: number, lat: number) {
		mapInstance?.flyTo({ center: [lng, lat], zoom: 14, speed: 1.8 });
	}
</script>

<button onclick={() => flyToCity(2.3522, 48.8566)}>Fly to Paris</button>

<Map bind:map={mapInstance} center={[0, 0]} zoom={2} />
```

Or inside any child component using context:

```svelte
<script lang="ts">
	import { useMap } from '$lib/components/ui/map';

	const { getMap, isLoaded } = useMap();

	function resetBearing() {
		getMap()?.resetNorthPitch();
	}
</script>
```

---

## 📚 API Reference

### `<Map />`

| Prop               | Type                                | Default                        | Description                                                                   |
| :----------------- | :---------------------------------- | :----------------------------- | :---------------------------------------------------------------------------- |
| `center`           | `[number, number]`                  | `[0, 0]`                       | Initial center coordinates `[lng, lat]`                                       |
| `zoom`             | `number`                            | `1`                            | Initial zoom level                                                            |
| `projection`       | `ProjectionSpecification`           | `{ type: "mercator" }`         | Map projection (`{ type: "globe" }` for 3D globe)                             |
| `styles`           | `{ light?: string; dark?: string }` | _Carto Positron / Dark Matter_ | Custom vector/raster style JSON URLs                                          |
| `blank`            | `boolean`                           | `false`                        | Renders a transparent basemap (ideal for custom GeoJSON visualizations)       |
| `theme`            | `"light" \| "dark"`                 | _Auto-detected_                | Explicit theme override                                                       |
| `options`          | `MapOptions`                        | `{}`                           | Additional MapLibre options (`pitch`, `maxPitch`, `attributionControl`, etc.) |
| `viewport`         | `Partial<MapViewport>`              | `undefined`                    | Controlled viewport driving center, zoom, bearing, and pitch                  |
| `bind:map`         | `MapLibreGL.Map \| null`            | `null`                         | Bindable reference to the underlying MapLibre instance                        |
| `onviewportchange` | `(v: MapViewport) => void`          | `undefined`                    | Continuous callback during pan, zoom, rotate, and tilt                        |
| `onstyleloaded`    | `() => void`                        | `undefined`                    | Callback fired when the active basemap style is fully loaded                  |

---

### `<MapMarker />`

| Prop             | Type                             | Default      | Description                                              |
| :--------------- | :------------------------------- | :----------- | :------------------------------------------------------- |
| `longitude`      | `number`                         | **Required** | Marker longitude coordinate                              |
| `latitude`       | `number`                         | **Required** | Marker latitude coordinate                               |
| `draggable`      | `boolean`                        | `false`      | Allows dragging the marker                               |
| `anchor`         | `Anchor`                         | `"center"`   | Marker alignment (`"center"`, `"top"`, `"bottom"`, etc.) |
| `offset`         | `PointLike`                      | `undefined`  | Pixel offset from the coordinate                         |
| `rotation`       | `number`                         | `0`          | Marker rotation in degrees                               |
| `pitchAlignment` | `"map" \| "viewport" \| "auto"`  | `"auto"`     | Marker pitch orientation                                 |
| `onclick`        | `(e: MouseEvent) => void`        | `undefined`  | Click event handler                                      |
| `ondragend`      | `(coords: { lng, lat }) => void` | `undefined`  | Drag end coordinates callback                            |

---

### `<MapArc />`

| Prop          | Type                              | Default      | Description                                                            |
| :------------ | :-------------------------------- | :----------- | :--------------------------------------------------------------------- |
| `data`        | `MapArcDatum[]`                   | **Required** | Array of `{ id, from: [lng, lat], to: [lng, lat] }`                    |
| `curvature`   | `number`                          | `0.2`        | Curve intensity (`0` = straight line, negative = bend other direction) |
| `samples`     | `number`                          | `64`         | Number of segments rendered along the curve                            |
| `paint`       | `LineLayerSpecification["paint"]` | `undefined`  | MapLibre paint settings (`line-color`, `line-width`, `line-dasharray`) |
| `hoverPaint`  | `LineLayerSpecification["paint"]` | `undefined`  | Styles applied when cursor hovers over an arc                          |
| `interactive` | `boolean`                         | `true`       | Enables hover and click detection                                      |
| `onclick`     | `(e: MapArcEvent) => void`        | `undefined`  | Arc click handler with coordinates and datum                           |

---

### `<MapRoute />`

| Prop          | Type                 | Default      | Description                                                        |
| :------------ | :------------------- | :----------- | :----------------------------------------------------------------- |
| `coordinates` | `[number, number][]` | **Required** | Array of waypoints defining the route line                         |
| `color`       | `string`             | `"#4285F4"`  | Line color                                                         |
| `width`       | `number`             | `3`          | Line width in pixels                                               |
| `opacity`     | `number`             | `0.8`        | Line opacity (0 to 1)                                              |
| `dashArray`   | `[number, number]`   | `undefined`  | Stroke dash pattern `[dash, gap]`                                  |
| `progress`    | `number`             | `undefined`  | Traveled fraction from `0` to `1` (drives progress layer & marker) |
| `active`      | `boolean`            | `false`      | Elevates route layer and applies active styling                    |

---

### `<MapClusterLayer />`

| Prop                | Type                          | Default             | Description                                     |
| :------------------ | :---------------------------- | :------------------ | :---------------------------------------------- |
| `data`              | `string \| FeatureCollection` | **Required**        | Point GeoJSON data or endpoint URL              |
| `clusterRadius`     | `number`                      | `50`                | Cluster radius in pixels                        |
| `clusterMaxZoom`    | `number`                      | `14`                | Max zoom level clustering is active             |
| `clusterColors`     | `[string, string, string]`    | `["#3b82f6", ...]`  | Colors for `[small, medium, large]` clusters    |
| `clusterThresholds` | `[number, number]`            | `[100, 750]`        | Count thresholds between cluster steps          |
| `onpointclick`      | `(feature, coords) => void`   | `undefined`         | Click handler for individual unclustered points |
| `onclusterclick`    | `(id, coords, count) => void` | _Zoom into cluster_ | Custom cluster click handler                    |

---

### `<MapControls />`

| Prop             | Type                                                           | Default          | Description                                   |
| :--------------- | :------------------------------------------------------------- | :--------------- | :-------------------------------------------- |
| `position`       | `"top-left" \| "top-right" \| "bottom-left" \| "bottom-right"` | `"bottom-right"` | Corner alignment                              |
| `showZoom`       | `boolean`                                                      | `true`           | Zoom in and zoom out buttons                  |
| `showCompass`    | `boolean`                                                      | `false`          | Dynamic 3D compass tracking pitch and bearing |
| `showLocate`     | `boolean`                                                      | `false`          | Geolocation button with live loading spinner  |
| `showFullscreen` | `boolean`                                                      | `false`          | Toggle browser fullscreen                     |
| `onlocate`       | `(coords) => void`                                             | `undefined`      | Fired when geolocation resolves coordinates   |

---

## 🎨 Basemap Styles

Mapcn includes clean, production-ready Carto basemaps by default. You can supply styles from any provider supporting the MapLibre/Mapbox GL style specification:

```ts
const basemaps = {
	// CARTO (Free, No Key Required for Low Traffic)
	cartoDark: 'https://basemaps.cartocdn.com/gl/dark-matter-nolabels-gl-style/style.json',
	cartoLight: 'https://basemaps.cartocdn.com/gl/positron-nolabels-gl-style/style.json',

	// MapTiler (Requires API Key)
	mapTilerDataviz: `https://api.maptiler.com/maps/dataviz/style.json?key=${API_KEY}`,

	// Stadia Maps (Requires API Key)
	stadiaAlidade: `https://tiles.stadiamaps.com/styles/alidade_smooth.json?api_key=${API_KEY}`
};
```

---

## 🛠️ Local Development

Clone the repository and run the demo app:

```bash
# Clone the repository
git clone https://github.com/nematollahi1999/Mapcn.git
cd Mapcn/map

# Install dependencies
pnpm install

# Start the dev server
pnpm dev
```

Open [http://localhost:5173](http://localhost:5173) in your browser to explore the interactive globe network demo.

### Scripts

- `pnpm dev` — Run Vite development server
- `pnpm build` — Build production bundle
- `pnpm check` — Run `svelte-check` type checking
- `pnpm lint` — Run ESLint and Prettier checks
- `pnpm format` — Auto-format code with Prettier

---

## 🤝 Contributing

Contributions are warmly welcome! Whether it's reporting a bug, proposing an API improvement, or submitting a new component:

1. Fork the repo
2. Create your feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

<div align="center">
  <sub>Built with ❤️ for the Svelte community. Inspired by <a href="https://shadcn-svelte.com">shadcn-svelte</a> and <a href="https://maplibre.org">MapLibre GL</a>.</sub>
</div>
