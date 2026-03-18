<script lang="ts">
	import { onMount } from 'svelte';
	import L from 'leaflet';
	import 'leaflet/dist/leaflet.css';
	import type { TrackPoint } from '$lib/gpxUtils';

	export let trackPoints: TrackPoint[] = [];
	export let height = '400px';
	let mapContainer: HTMLDivElement;
	let map: L.Map | null = null;
	let polyline: L.Polyline | null = null;

	onMount(() => {
		if (trackPoints.length > 0) {
			initializeMap();
		}
	});

	function initializeMap() {
		if (!trackPoints || trackPoints.length === 0) return;

		const latLngs: [number, number][] = trackPoints.map((p) => [p.lat, p.lon]);

		map = L.map(mapContainer, {
			scrollWheelZoom: false
		});

		L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
			attribution: '&copy; OpenStreetMap contributors',
			maxZoom: 19
		}).addTo(map);

		const colors = ['#2196F3', '#4CAF50', '#FF9800', '#9C27B0', '#F44336'];
		const colorIndex = Math.floor(Math.random() * colors.length);
		const routeColor = colors[colorIndex];

		polyline = L.polyline(latLngs, {
			color: routeColor,
			weight: 5,
			opacity: 0.8,
			smoothFactor: 1
		}).addTo(map);

		const bounds = polyline.getBounds();
		map.fitBounds(bounds, { padding: [50, 50] });

		if (trackPoints.length > 0) {
			const startMarker = L.circleMarker([trackPoints[0].lat, trackPoints[0].lon], {
				radius: 8,
				fillColor: '#4CAF50',
				color: '#fff',
				weight: 2,
				opacity: 1,
				fillOpacity: 1
			}).addTo(map);

			startMarker.bindTooltip('Start', { permanent: false, direction: 'top' });

			const endPoint = trackPoints[trackPoints.length - 1];
			const endMarker = L.circleMarker([endPoint.lat, endPoint.lon], {
				radius: 8,
				fillColor: '#F44336',
				color: '#fff',
				weight: 2,
				opacity: 1,
				fillOpacity: 1
			}).addTo(map);

			endMarker.bindTooltip('End', { permanent: false, direction: 'top' });
		}
	}

	export function updateMap(newPoints: TrackPoint[]) {
		if (!newPoints || newPoints.length === 0) return;

		trackPoints = newPoints;

		if (polyline && map) {
			map.removeLayer(polyline);
		}

		const latLngs: [number, number][] = trackPoints.map((p) => [p.lat, p.lon]);
		const colors = ['#2196F3', '#4CAF50', '#FF9800', '#9C27B0', '#F44336'];
		const colorIndex = Math.floor(Math.random() * colors.length);
		const routeColor = colors[colorIndex];

		polyline = L.polyline(latLngs, {
			color: routeColor,
			weight: 5,
			opacity: 0.8,
			smoothFactor: 1
		}).addTo(map!);

		const bounds = polyline.getBounds();
		map!.fitBounds(bounds, { padding: [50, 50] });

		const markers = map!.eachLayer((layer: L.Layer) => {
			if (layer instanceof L.CircleMarker) {
				map!.removeLayer(layer);
			}
		});

		if (trackPoints.length > 0) {
			const startMarker = L.circleMarker([trackPoints[0].lat, trackPoints[0].lon], {
				radius: 8,
				fillColor: '#4CAF50',
				color: '#fff',
				weight: 2,
				opacity: 1,
				fillOpacity: 1
			}).addTo(map!);

			startMarker.bindTooltip('Start', { permanent: false, direction: 'top' });

			const endPoint = trackPoints[trackPoints.length - 1];
			const endMarker = L.circleMarker([endPoint.lat, endPoint.lon], {
				radius: 8,
				fillColor: '#F44336',
				color: '#fff',
				weight: 2,
				opacity: 1,
				fillOpacity: 1
			}).addTo(map!);

			endMarker.bindTooltip('End', { permanent: false, direction: 'top' });
		}
	}

	$: if (map && trackPoints.length > 0) {
		setTimeout(() => {
			map!.invalidateSize();
		}, 100);
	}
</script>

<div
	bind:this={mapContainer}
	class="gpx-map"
	style="height: {height}; width: 100%; min-height: 300px;"
></div>

<style>
	:global(.gpx-map .leaflet-container) {
		border-radius: 0.5rem;
		border: 1px solid #e5e7eb;
	}
</style>
