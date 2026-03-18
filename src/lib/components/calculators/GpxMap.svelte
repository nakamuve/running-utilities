<script lang="ts">
	import { onMount, tick } from 'svelte';
	import L from 'leaflet';
	import 'leaflet/dist/leaflet.css';
	import type { TrackPoint, WayPoint } from '$lib/gpxUtils';

	export let trackPoints: TrackPoint[] = [];
	export let wayPoints: WayPoint[] = [];
	export let height = '400px';
	let mapContainer: HTMLDivElement;
	let map: L.Map | null = null;
	let polyline: L.Polyline | null = null;
	let isMounted = false;

	onMount(() => {
		isMounted = true;
		if (trackPoints.length > 0 && mapContainer) {
			initializeMap();
		}
	});

	function initializeMap() {
		if (!trackPoints || trackPoints.length === 0 || !mapContainer) return;

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
		map!.fitBounds(bounds, { padding: [50, 50] });

		if (wayPoints && wayPoints.length > 0) {
			wayPoints.forEach((wp) => {
				const marker = L.circleMarker([wp.lat, wp.lon], {
					radius: 10,
					fillColor: '#FF9800',
					color: '#fff',
					weight: 3,
					opacity: 1,
					fillOpacity: 1
				}).addTo(map!);

				marker.bindTooltip(wp.name || 'Checkpoint', { permanent: false, direction: 'top' });
			});
		}

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

		if (polyline && map) {
			map.removeLayer(polyline);
		}

		const latLngs: [number, number][] = newPoints.map((p) => [p.lat, p.lon]);
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

		if (newPoints.length > 0) {
			const startMarker = L.circleMarker([newPoints[0].lat, newPoints[0].lon], {
				radius: 8,
				fillColor: '#4CAF50',
				color: '#fff',
				weight: 2,
				opacity: 1,
				fillOpacity: 1
			}).addTo(map!);

			startMarker.bindTooltip('Start', { permanent: false, direction: 'top' });

			const endPoint = newPoints[newPoints.length - 1];
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

		if (wayPoints && wayPoints.length > 0) {
			wayPoints.forEach((wp) => {
				const marker = L.circleMarker([wp.lat, wp.lon], {
					radius: 10,
					fillColor: '#FF9800',
					color: '#fff',
					weight: 3,
					opacity: 1,
					fillOpacity: 1
				}).addTo(map!);

				marker.bindTooltip(wp.name || 'Checkpoint', { permanent: false, direction: 'top' });
			});
		}
	}

	$: if (isMounted && trackPoints.length > 0) {
		(async () => {
			await tick();
			if (map) {
				updateMap(trackPoints);
			} else if (mapContainer) {
				initializeMap();
			}
		})();
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
