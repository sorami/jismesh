<script lang="ts">
	import '@unocss/reset/tailwind-compat.css';
	import 'virtual:uno.css';
	import 'maplibre-gl/dist/maplibre-gl.css';
	import { Map, LngLat, NavigationControl, GeolocateControl, GeoJSONSource } from 'maplibre-gl';
	// @ts-ignore
	import * as jismesh from 'jismesh-js';
	import { onMount } from 'svelte';
	import type { GeoJSON } from 'geojson';

	const INITIAL_CENTER = new LngLat(139.741357, 35.658099); // 日本経緯度原点
	const INITIAL_ZOOM = 8;

	let center = INITIAL_CENTER;
	let zoom = INITIAL_ZOOM;

	let map: Map | undefined;

	const degrees: { id: number; text: string }[] = [
		{ id: 1, text: '1次' },
		{ id: 2, text: '2次' },
		{ id: 3, text: '3次' },
		{ id: 4, text: '4次' },
		{ id: 5, text: '5次' }
	];
	let selectedDegree: number = 1;

	let meshcode: number;

	function update(center: LngLat, zoom: number) {
		meshcode = jismesh.toMeshCode(center.lat, center.lng, selectedDegree);

		// 南西端
		const [lat0, lon0] = jismesh.toMeshPoint(meshcode, 0, 0);
		// 北東端
		const [lat1, lon1] = jismesh.toMeshPoint(meshcode, 1, 1);

		const polygonData: GeoJSON = {
			type: 'Feature',
			geometry: {
				type: 'Polygon',
				coordinates: [
					[
						[lon0, lat0],
						[lon1, lat0],
						[lon1, lat1],
						[lon0, lat1],
						[lon0, lat0]
					]
				]
			},
			properties: {
				meshcode
			}
		};

		if (map) {
			const source = map.getSource('mesh') as GeoJSONSource;
			if (source) source.setData(polygonData);
		}
	}

	$: {
		selectedDegree;
		update(center, zoom);
	}

	onMount(() => {
		map = new Map({
			container: 'map',
			style: 'https://demotiles.maplibre.org/style.json',
			center: INITIAL_CENTER,
			zoom: INITIAL_ZOOM
		});

		map.addControl(new NavigationControl());
		map.addControl(new GeolocateControl({}));

		map.on('load', () => {
			if (!map) return;

			map.addSource('mesh', {
				type: 'geojson',
				data: {
					type: 'Feature',
					geometry: {},
					properties: {}
				}
			});
			map.addLayer({
				id: 'mesh-polygon',
				type: 'fill',
				source: 'mesh',
				layout: {},
				paint: {
					'fill-color': '#333',
					'fill-opacity': 0.8
				}
			});

			map.addLayer({
				id: 'mesh-label',
				type: 'symbol',
				source: 'mesh',
				layout: {
					'text-field': ['get', 'meshcode'], // property in your GeoJSON containing the label text
					'text-size': 48,
					'text-anchor': 'center'
				},
				paint: {
					'text-color': '#fff'
				}
			});

			center = map.getCenter();
			zoom = map.getZoom();
			update(center, zoom);
		});

		map.on('move', () => {
			if (!map) return;
			center = map.getCenter();
			zoom = map.getZoom();
		});
	});
</script>

<div class="fixed left-8 top-8 z-10 bg-black rounded p-6 text-white opacity-70">
	<div>緯度: {center.lat.toFixed(4)}</div>
	<div>経度: {center.lng.toFixed(4)}</div>
	<div>ズーム: {zoom.toFixed(2)}</div>

	<br />

	<h2>メッシュ</h2>
	<form on:submit|preventDefault={() => {}}>
		<select name="mesh-degree" bind:value={selectedDegree} class="w-full text-black rounded">
			{#each degrees as degree (degree.id)}
				<option value={degree.id} class="text-black">{degree.text}</option>
			{/each}
		</select>
	</form>
	<div>meshcode: {meshcode}</div>
</div>
<div id="map" class="w-screen h-screen" />
