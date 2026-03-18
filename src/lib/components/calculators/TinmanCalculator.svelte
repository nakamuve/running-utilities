<script>
	import { tinmanCalculate, tinmanDistanceOptions, timeToSeconds } from '$lib/calculatorUtils';

	// Define state for form data
	let formData = $state({
		raceDistance: '5k', // Default to 5k
		hours: 0,
		minutes: 0,
		seconds: 0
	});

	let calculatedPaces = $state(null);
	let errorMessage = $state('');

	function calculatePaces() {
		try {
			// Convert time to seconds
			const totalSeconds = timeToSeconds(
				parseInt(formData.hours) || 0,
				parseInt(formData.minutes) || 0,
				parseInt(formData.seconds) || 0
			);

			if (isNaN(totalSeconds) || totalSeconds <= 0) {
				errorMessage = 'Please enter a valid time';
				return;
			}

			// Use the existing tinmanCalculate function from calculatorUtils.ts
			calculatedPaces = tinmanCalculate(formData.raceDistance, totalSeconds);
			errorMessage = '';
		} catch (error) {
			errorMessage = `Error calculating paces: ${error.message}`;
			calculatedPaces = null;
		}
	}
</script>

<div class="rounded-lg bg-white p-6 shadow-md">
	<h1 class="mb-4 text-xl font-bold">Tinman Running Calculator</h1>
	<p class="mb-4 text-gray-600">
		Calculate equivalent race times and training paces based on Tom Schwartz's Critical Velocity
		(CV) methodology.
	</p>

	<div class="mb-4">
		<label class="mb-2 block text-gray-700">Race Distance</label>
		<select bind:value={formData.raceDistance} class="w-full rounded border p-2">
			{#each tinmanDistanceOptions as option}
				<option value={option.value}>{option.label}</option>
			{/each}
		</select>
	</div>

	<div class="mb-4">
		<label class="mb-2 block text-gray-700">Race Time</label>
		<div class="grid grid-cols-3 gap-4">
			<div>
				<label class="block text-sm text-gray-600">Hours</label>
				<input
					type="number"
					bind:value={formData.hours}
					class="w-full rounded border p-2"
					min="0"
					placeholder="HH"
				/>
			</div>
			<div>
				<label class="block text-sm text-gray-600">Minutes</label>
				<input
					type="number"
					bind:value={formData.minutes}
					class="w-full rounded border p-2"
					min="0"
					max="59"
					placeholder="MM"
				/>
			</div>
			<div>
				<label class="block text-sm text-gray-600">Seconds</label>
				<input
					type="number"
					bind:value={formData.seconds}
					class="w-full rounded border p-2"
					min="0"
					max="59"
					placeholder="SS"
				/>
			</div>
		</div>
	</div>

	<button
		class="rounded bg-blue-500 px-4 py-2 font-bold text-white hover:bg-blue-600"
		on:click={calculatePaces}
	>
		Calculate Paces
	</button>

	{#if errorMessage}
		<div class="my-4 border-l-4 border-red-500 bg-red-100 p-4 text-red-700" role="alert">
			<p>{errorMessage}</p>
		</div>
	{/if}

	{#if calculatedPaces}
		<div class="mt-6">
			<h2 class="text-lg font-bold">Results</h2>

			<div class="mt-4">
				<h3 class="font-bold">Critical Velocity (CV) Pace</h3>
				<p class="text-gray-700">{calculatedPaces.cv_pace} per km</p>
			</div>

			<div class="mt-4">
				<h3 class="font-bold">Equivalent Race Times</h3>
				<div class="mt-2 overflow-x-auto">
					<table class="min-w-full border bg-white">
						<thead>
							<tr>
								<th class="border-b border-gray-200 bg-gray-50 px-4 py-2 text-left">Distance</th>
								<th class="border-b border-gray-200 bg-gray-50 px-4 py-2 text-left"
									>Predicted Time</th
								>
							</tr>
						</thead>
						<tbody>
							{#each Object.entries(calculatedPaces.equivalent_races) as [distance, time]}
								<tr>
									<td class="border-b border-gray-200 px-4 py-2">
										{tinmanDistanceOptions.find((option) => option.value === distance)?.label ||
											distance}
									</td>
									<td class="border-b border-gray-200 px-4 py-2">{time}</td>
								</tr>
							{/each}
						</tbody>
					</table>
				</div>
			</div>

			<div class="mt-4">
				<h3 class="font-bold">Training Paces (per km)</h3>
				<div class="mt-2 overflow-x-auto">
					<table class="min-w-full border bg-white">
						<thead>
							<tr>
								<th class="border-b border-gray-200 bg-gray-50 px-4 py-2 text-left"
									>Training Type</th
								>
								<th class="border-b border-gray-200 bg-gray-50 px-4 py-2 text-left">Pace Range</th>
							</tr>
						</thead>
						<tbody>
							{#each Object.entries(calculatedPaces.training_paces) as [type, range]}
								<tr>
									<td class="border-b border-gray-200 px-4 py-2">
										{type
											.split('_')
											.map((word) => word.charAt(0).toUpperCase() + word.slice(1))
											.join(' ')}
									</td>
									<td class="border-b border-gray-200 px-4 py-2">{range.min} - {range.max}</td>
								</tr>
							{/each}
						</tbody>
					</table>
				</div>
			</div>
		</div>
	{/if}
</div>
