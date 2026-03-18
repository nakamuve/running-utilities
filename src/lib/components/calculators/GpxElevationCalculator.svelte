<script>
	import { analyzeGpxElevation } from '$lib/gpxUtils';
	import GpxMap from './GpxMap.svelte';

	let uploadedFile = null;
	let loading = false;
	let error = '';
	let analysisResults = null;
	let segmentDistance = 1000; // Default: 1km (1000m)

	// Handle file selection change
	function handleFileSelect(event) {
		const fileInput = event.target;
		if (fileInput.files.length > 0) {
			uploadedFile = fileInput.files[0];
			error = '';
		} else {
			uploadedFile = null;
		}
	}

	// Handle file upload and analysis
	async function uploadAndAnalyzeFile() {
		if (!uploadedFile) {
			error = 'Please select a GPX file';
			return;
		}

		if (!uploadedFile.name.toLowerCase().endsWith('.gpx')) {
			error = 'Please select a valid GPX file';
			return;
		}

		loading = true;
		error = '';

		try {
			const reader = new FileReader();

			// Create a promise for the file reading operation
			const fileContent = await new Promise((resolve, reject) => {
				reader.onload = (e) => resolve(e.target.result);
				reader.onerror = () => reject(new Error('Error reading file'));
				reader.readAsText(uploadedFile);
			});

			// Analyze the GPX file with selected segment distance
			const results = analyzeGpxElevation(
				fileContent.toString(),
				uploadedFile.name,
				segmentDistance
			);

			if (results.error) {
				throw new Error(results.error);
			}

			// Calculate additional gradient statistics
			if (results.km_segments && results.km_segments.length > 0) {
				const gradients = results.km_segments.map((segment) => segment.grade);

				// Sort gradients for median and min/max
				const sortedGradients = [...gradients].sort((a, b) => a - b);

				// Calculate median gradient
				const mid = Math.floor(sortedGradients.length / 2);
				const medianGradient =
					sortedGradients.length % 2 !== 0
						? sortedGradients[mid]
						: (sortedGradients[mid - 1] + sortedGradients[mid]) / 2;

				// Calculate min and max gradient
				const minGradient = sortedGradients[0];
				const maxGradient = sortedGradients[sortedGradients.length - 1];

				// Add these to the results
				results.median_gradient = Math.round(medianGradient * 10) / 10;
				results.min_gradient = Math.round(minGradient * 10) / 10;
				results.max_gradient = Math.round(maxGradient * 10) / 10;
			}

			analysisResults = results;
		} catch (err) {
			error = `Error analyzing file: ${err.message}`;
			console.error(error);
			analysisResults = null;
		} finally {
			loading = false;
		}
	}

	// Helper to format elevation values
	function formatElevation(value) {
		return value.toFixed(1);
	}

	// Handle exporting data to CSV
	function exportToCsv() {
		if (!analysisResults) return;

		const segmentLabel = segmentDistance === 500 ? 'Segment (500m)' : 'Kilometer';
		let csvContent = `${segmentLabel},Current Elevation (m),Elevation Gain (m),Elevation Loss (m),Net Elevation (m),Grade (%)\n`;

		// Add table data rows
		for (const segment of analysisResults.km_segments) {
			const segmentName =
				typeof segment.km === 'number' ? getFormattedSegmentLabel(segment.km) : segment.km;
			csvContent += `${segmentName},${segment.current_elevation},${segment.elevation_gain},${segment.elevation_loss},${segment.net_elevation},${segment.grade}\n`;
		}

		// Add summary data
		csvContent += '\n';
		csvContent += `Summary,,"${analysisResults.total_elevation_gain}","${analysisResults.total_elevation_loss}","${analysisResults.net_elevation}",\n`;
		csvContent += `Total Distance,${analysisResults.total_distance} km\n`;
		csvContent += `Start Elevation,${analysisResults.start_elevation} m\n`;
		csvContent += `End Elevation,${analysisResults.end_elevation} m\n`;
		csvContent += `Average Gain/km,${analysisResults.avg_elevation_gain_per_km} m\n`;
		csvContent += `Average Gradient,${analysisResults.overall_grade}%\n`;
		csvContent += `Median Gradient,${analysisResults.median_gradient}%\n`;
		csvContent += `Minimum Gradient,${analysisResults.min_gradient}%\n`;
		csvContent += `Maximum Gradient,${analysisResults.max_gradient}%\n`;

		// Create a blob and download link
		const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
		const url = URL.createObjectURL(blob);

		// Create a temporary link and trigger download
		const link = document.createElement('a');
		link.href = url;
		link.setAttribute('download', `${analysisResults.filename || 'elevation-data'}.csv`);
		link.style.display = 'none';
		document.body.appendChild(link);
		link.click();

		// Clean up
		document.body.removeChild(link);
		URL.revokeObjectURL(url);
	}

	// Calculate segment label for display
	function getSegmentLabel() {
		return segmentDistance === 500 ? 'Segment (500m)' : 'Kilometer';
	}

	// Format segment label with distance information
	function getFormattedSegmentLabel(segmentNum) {
		if (segmentDistance === 500) {
			// Calculate the start and end distance for this segment in km
			const startKm = (segmentNum - 1) * 0.5;
			const endKm = segmentNum * 0.5;
			return `${startKm.toFixed(1)}-${endKm.toFixed(1)} km`;
		} else {
			// For 1km segments, show the distance range
			const startKm = segmentNum - 1;
			const endKm = segmentNum;
			return `${startKm}-${endKm} km`;
		}
	}
</script>

<div
	class="gpx-elevation-calculator mb-8 rounded-lg bg-white p-6 shadow-md"
	id="gpx-calculator-form"
>
	<h2 class="mb-4 text-xl font-semibold">GPX Elevation Gain Calculator</h2>

	<div class="mb-4">
		<p class="mb-4 text-gray-700">
			Upload a GPX file to analyze elevation gain per kilometer. This tool calculates the elevation
			gain and loss for each kilometer segment, allowing you to understand the difficulty of your
			route.
		</p>

		{#if error}
			<div role="alert" class="alert alert-error mb-4">
				<svg
					xmlns="http://www.w3.org/2000/svg"
					class="h-6 w-6 shrink-0 stroke-current"
					fill="none"
					viewBox="0 0 24 24"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M10 14l2-2m0 0l2-2m-2 2l-2-2m2 2l2 2m7-2a9 9 0 11-18 0 9 9 0 0118 0z"
					/>
				</svg>
				<span>{error}</span>
			</div>
		{/if}

		<!-- Segment distance option -->
		<div class="mb-4">
			<label class="form-control">
				<div class="label">
					<span class="label-text font-medium">Segment Distance</span>
				</div>
				<div class="join">
					<label class="join-item btn {segmentDistance === 1000 ? 'btn-active' : ''}">
						<input
							type="radio"
							name="segment-distance"
							class="hidden"
							checked={segmentDistance === 1000}
							on:click={() => (segmentDistance = 1000)}
							aria-label="1 Kilometer"
						/>
						1 km
					</label>
					<label class="join-item btn {segmentDistance === 500 ? 'btn-active' : ''}">
						<input
							type="radio"
							name="segment-distance"
							class="hidden"
							checked={segmentDistance === 500}
							on:click={() => (segmentDistance = 500)}
							aria-label="500 Meters"
						/>
						500 m
					</label>
				</div>
			</label>
		</div>

		<!-- File Input and Submit Button - Responsive Layout -->
		<div class="mb-4 flex flex-col gap-4 md:flex-row">
			<div class="form-control w-full">
				<div class="join w-full flex-col md:flex-row">
					<input
						type="file"
						class="join-item file-input file-input-bordered w-full"
						accept=".gpx"
						on:change={handleFileSelect}
					/>
					<button
						class="join-item btn btn-primary mt-2 md:mt-0"
						class:btn-disabled={loading || !uploadedFile}
						on:click={uploadAndAnalyzeFile}
						disabled={loading || !uploadedFile}
					>
						<span>Analyze Elevation Gain</span>
						{#if loading}
							<span class="loading loading-spinner loading-sm"></span>
						{/if}
					</button>
				</div>
			</div>
		</div>
	</div>

	{#if analysisResults}
		<div class="mt-8 mb-8 rounded-lg bg-white p-6 shadow-md">
			<div class="mb-4 flex items-center justify-between">
				<h2 class="text-xl font-semibold">
					GPX Analysis Results: {analysisResults.filename || 'Uploaded File'}
				</h2>
				<button id="export-csv-btn" class="btn btn-primary btn-sm gap-2" on:click={exportToCsv}>
					<svg
						xmlns="http://www.w3.org/2000/svg"
						class="h-5 w-5"
						fill="none"
						viewBox="0 0 24 24"
						stroke="currentColor"
					>
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"
						/>
					</svg>
					Export CSV
				</button>
			</div>

			<!-- Summary Statistics -->
			<div class="mb-6 grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-4">
				<div class="rounded-lg bg-blue-50 p-4">
					<p class="text-sm text-gray-600">Total Distance</p>
					<p class="text-2xl font-bold">{analysisResults.total_distance} km</p>
				</div>
				<div class="rounded-lg bg-green-50 p-4">
					<p class="text-sm text-gray-600">Total Elevation Gain</p>
					<p class="text-2xl font-bold">{analysisResults.total_elevation_gain} m</p>
				</div>
				<div class="rounded-lg bg-red-50 p-4">
					<p class="text-sm text-gray-600">Total Elevation Loss</p>
					<p class="text-2xl font-bold">{analysisResults.total_elevation_loss} m</p>
				</div>
				<div class="rounded-lg bg-purple-50 p-4">
					<p class="text-sm text-gray-600">Avg. Elevation Gain per km</p>
					<p class="text-2xl font-bold">{analysisResults.avg_elevation_gain_per_km} m</p>
				</div>
			</div>

			<!-- Start/End Elevation -->
			<div class="mb-6 grid grid-cols-1 gap-4 md:grid-cols-2 lg:grid-cols-4">
				<div class="rounded-lg bg-indigo-50 p-4">
					<p class="text-sm text-gray-600">Starting Elevation</p>
					<p class="text-2xl font-bold">{analysisResults.start_elevation} m</p>
				</div>
				<div class="rounded-lg bg-indigo-50 p-4">
					<p class="text-sm text-gray-600">Ending Elevation</p>
					<p class="text-2xl font-bold">{analysisResults.end_elevation} m</p>
				</div>
				<div class="rounded-lg bg-yellow-50 p-4">
					<p class="text-sm text-gray-600">Average Gradient</p>
					<p class="text-2xl font-bold">{analysisResults.overall_grade}%</p>
				</div>
				<div class="rounded-lg bg-yellow-50 p-4">
					<p class="text-sm text-gray-600">Median Gradient</p>
					<p class="text-2xl font-bold">{analysisResults.median_gradient}%</p>
				</div>
			</div>

			<!-- Min/Max Gradient -->
			<div class="mb-6 grid grid-cols-1 gap-4 md:grid-cols-2">
				<div class="rounded-lg bg-orange-50 p-4">
					<p class="text-sm text-gray-600">Minimum Gradient</p>
					<p class="text-2xl font-bold">{analysisResults.min_gradient}%</p>
				</div>
				<div class="rounded-lg bg-orange-50 p-4">
					<p class="text-sm text-gray-600">Maximum Gradient</p>
					<p class="text-2xl font-bold">{analysisResults.max_gradient}%</p>
				</div>
			</div>

			<!-- Route Map -->
			{#if analysisResults.track_points && analysisResults.track_points.length > 0}
				<h3 class="mb-4 text-lg font-semibold">Route Map</h3>
				<div class="mb-6">
					<GpxMap
						trackPoints={analysisResults.track_points}
						wayPoints={analysisResults.waypoints || []}
						height="500px"
					/>
				</div>
			{/if}

			<!-- Waypoints Table -->
			{#if analysisResults.waypoints && analysisResults.waypoints.length > 0}
				<h3 class="mb-4 text-lg font-semibold">Waypoints</h3>
				<div class="mb-6 overflow-x-auto">
					<table class="table-md table w-full border">
						<thead class="bg-base-200 text-base-content">
							<tr>
								<th class="bg-base-300">Waypoint Name</th>
								<th class="text-right">Distance from Start</th>
							</tr>
						</thead>
						<tbody>
							{#each analysisResults.waypoints.sort((a, b) => a.distance_from_start - b.distance_from_start) as waypoint}
								<tr class="hover">
									<td class="bg-base-100 font-medium">
										{waypoint.name || 'Unnamed'}
									</td>
									<td class="text-right">
										{(waypoint.distance_from_start / 1000).toFixed(2)} km
									</td>
								</tr>
							{/each}
						</tbody>
					</table>
				</div>
			{/if}

			<!-- Kilometer Breakdown -->
			<h3 class="mb-4 text-lg font-semibold">Elevation Gain Per {getSegmentLabel()}</h3>

			<div class="overflow-x-auto">
				<table class="table-md table w-full border" id="elevation-data-table">
					<thead class="bg-base-200 text-base-content">
						<tr>
							<th class="bg-base-300">{getSegmentLabel()}</th>
							<th class="text-right">Current Elevation (m)</th>
							<th class="text-right">Elevation Gain (m)</th>
							<th class="text-right">Elevation Loss (m)</th>
							<th class="text-right">Net Elevation (m)</th>
							<th class="text-right">Grade (%)</th>
						</tr>
					</thead>
					<tbody>
						{#each analysisResults.km_segments as segment}
							<tr class="hover">
								<td class="bg-base-100 font-medium">
									{typeof segment.km === 'number'
										? getFormattedSegmentLabel(segment.km)
										: segment.km}
								</td>
								<td class="text-right">{segment.current_elevation}</td>
								<td
									class="text-right"
									class:text-success={segment.elevation_gain >
										analysisResults.avg_elevation_gain_per_km}
									class:font-medium={segment.elevation_gain >
										analysisResults.avg_elevation_gain_per_km}
								>
									{segment.elevation_gain}
								</td>
								<td
									class="text-right"
									class:text-error={segment.elevation_loss >
										analysisResults.avg_elevation_gain_per_km}
									class:font-medium={segment.elevation_loss >
										analysisResults.avg_elevation_gain_per_km}
								>
									{segment.elevation_loss}
								</td>
								<td
									class="text-right"
									class:text-success={segment.net_elevation > 0}
									class:text-error={segment.net_elevation < 0}
									class:font-medium={segment.net_elevation != 0}
								>
									{segment.net_elevation}
								</td>
								<td
									class="text-right"
									class:text-success={segment.grade > 5}
									class:text-error={segment.grade < -5}
									class:text-bold={segment.grade > 5 || segment.grade < -5}
								>
									{segment.grade}%
								</td>
							</tr>
						{/each}
					</tbody>
				</table>
			</div>

			<!-- Visualization Info -->
			<div class="alert alert-info mt-6">
				<svg
					xmlns="http://www.w3.org/2000/svg"
					fill="none"
					viewBox="0 0 24 24"
					class="h-6 w-6 shrink-0 stroke-current"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
					></path>
				</svg>
				<div>
					<h3 class="font-bold">Want more detailed analysis?</h3>
					<div class="text-sm">
						For more advanced visualizations and analysis, consider using tools like:
					</div>
					<ul class="mt-2 list-inside list-disc text-sm">
						<li>
							<a href="https://gpx.studio" class="link link-primary" target="_blank" rel="noopener"
								>GPX Studio</a
							> - Interactive GPX editor and analyzer
						</li>
						<li>
							<a
								href="https://www.strava.com"
								class="link link-primary"
								target="_blank"
								rel="noopener">Strava</a
							> - Advanced activity analysis
						</li>
					</ul>
				</div>
			</div>
		</div>
	{/if}
</div>
