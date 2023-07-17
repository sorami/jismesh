<script lang="ts">
	import '@unocss/reset/tailwind-compat.css';
	import 'virtual:uno.css';
	import 'maplibre-gl/dist/maplibre-gl.css';
	import maplibre from 'maplibre-gl';
	import { onMount } from 'svelte';

	onMount(() => {
		const map = new maplibre.Map({
			container: 'map',
			style: 'https://demotiles.maplibre.org/style.json',
			center: [139.741357, 35.658099], // 日本経緯度原点
			zoom: 5
		});

		map.addControl(new maplibre.NavigationControl());
		map.addControl(new maplibre.GeolocateControl({}));
		map.addControl(new maplibre.FullscreenControl());
	});
</script>

<div id="map" class="w-screen h-screen" />
