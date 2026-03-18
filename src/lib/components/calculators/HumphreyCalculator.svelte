<script context="module">
	function formatDistanceLabel(distance) {
		switch (distance) {
			case 'mile':
				return 'Mile';
			case '2miles':
				return '2 Miles';
			case '3k':
				return '3K';
			case '4k':
				return '4K';
			case '5k':
				return '5K';
			case '6k':
				return '6K';
			case '8k':
				return '8K';
			case '10k':
				return '10K';
			case '12k':
				return '12K';
			case '15k':
				return '15K';
			case '10miles':
				return '10 Miles';
			case '20k':
				return '20K';
			case 'half_marathon':
				return 'Half Marathon';
			case '25k':
				return '25K';
			case '30k':
				return '30K';
			case 'marathon':
				return 'Marathon';
			default:
				return distance;
		}
	}

	function formatTrainingType(type) {
		return type
			.split('_')
			.map((word) => word.charAt(0).toUpperCase() + word.slice(1))
			.join(' ');
	}
</script>

<script>
	import { humphreyCalculate, timeToSeconds, humphreyDistanceOptions } from '$lib/calculatorUtils';

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
			// Clear previous error
			errorMessage = '';

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

			// Use the humphreyCalculate function
			calculatedPaces = humphreyCalculate(formData.raceDistance, totalSeconds);
		} catch (error) {
			errorMessage = `Error calculating paces: ${error.message}`;
			calculatedPaces = null;
		}
	}
</script>

<div class="rounded-lg bg-white p-6 shadow-md">
	<h1 class="text-primary mb-4 text-2xl font-bold">Hansons/Humphrey Running Calculator</h1>
	<p class="mb-6 text-gray-600">
		Calculate equivalent race times and training paces based on the Hansons Method.
	</p>

	<div class="mb-8 border-b pb-6">
		<h2 class="text-primary mb-4 text-lg font-bold">Input Data</h2>

		<div class="mb-4">
			<label class="mb-2 block font-semibold text-gray-700" for="race-distance">Race Distance</label
			>
			<select
				id="race-distance"
				bind:value={formData.raceDistance}
				class="focus:border-primary focus:ring-primary w-full rounded border p-2 focus:ring-1"
			>
				{#each humphreyDistanceOptions as option}
					<option value={option.value}>{option.label}</option>
				{/each}
			</select>
		</div>

		<div class="mb-4">
			<label class="mb-2 block font-semibold text-gray-700">Race Time</label>
			<div class="grid grid-cols-3 gap-4">
				<div>
					<label class="mb-1 block text-sm text-gray-600">Hours</label>
					<input
						type="number"
						bind:value={formData.hours}
						class="focus:border-primary focus:ring-primary w-full rounded border p-2 focus:ring-1"
						min="0"
						placeholder="HH"
					/>
				</div>
				<div>
					<label class="mb-1 block text-sm text-gray-600">Minutes</label>
					<input
						type="number"
						bind:value={formData.minutes}
						class="focus:border-primary focus:ring-primary w-full rounded border p-2 focus:ring-1"
						min="0"
						max="59"
						placeholder="MM"
					/>
				</div>
				<div>
					<label class="mb-1 block text-sm text-gray-600">Seconds</label>
					<input
						type="number"
						bind:value={formData.seconds}
						class="focus:border-primary focus:ring-primary w-full rounded border p-2 focus:ring-1"
						min="0"
						max="59"
						placeholder="SS"
					/>
				</div>
			</div>
		</div>

		{#if errorMessage}
			<div class="mb-4 rounded border border-red-400 bg-red-100 px-4 py-3 text-red-700">
				<p>{errorMessage}</p>
			</div>
		{/if}

		<div class="flex justify-end">
			<button
				class="bg-primary hover:bg-primary-focus focus:shadow-outline rounded px-6 py-2 font-bold text-white transition duration-200 focus:outline-none"
				on:click={calculatePaces}
			>
				Calculate
			</button>
		</div>
	</div>

	{#if calculatedPaces}
		<div class="mt-8">
			<h2 class="text-primary mb-6 text-xl font-bold">Results</h2>

			<div class="bg-base-100 mb-8 rounded-lg p-4 shadow-sm">
				<h3 class="text-primary mb-3 text-lg font-bold">Race Information</h3>
				<div class="grid grid-cols-1 gap-4 md:grid-cols-3">
					<div class="bg-base-200 rounded p-3">
						<span class="block text-sm text-gray-500">Distance</span>
						<span class="text-lg font-medium"
							>{formatDistanceLabel(calculatedPaces.race_distance)}</span
						>
					</div>
					<div class="bg-base-200 rounded p-3">
						<span class="block text-sm text-gray-500">Time</span>
						<span class="text-lg font-medium">{calculatedPaces.race_time}</span>
					</div>
					<div class="bg-base-200 rounded p-3">
						<span class="block text-sm text-gray-500">Pace per km</span>
						<span class="text-lg font-medium">{calculatedPaces.pace_per_km}</span>
					</div>
				</div>
			</div>

			<div class="mb-8">
				<h3 class="text-primary mb-3 text-lg font-bold">Equivalent Race Times</h3>
				<div class="bg-base-100 overflow-x-auto rounded-lg shadow-sm">
					<table class="table w-full">
						<thead>
							<tr>
								<th
									class="bg-base-200 px-4 py-3 text-left text-xs font-medium tracking-wider text-gray-500 uppercase"
									>Distance</th
								>
								<th
									class="bg-base-200 px-4 py-3 text-left text-xs font-medium tracking-wider text-gray-500 uppercase"
									>Predicted Time</th
								>
							</tr>
						</thead>
						<tbody>
							{#each Object.entries(calculatedPaces.equivalent_races) as [distance, time]}
								<tr class="hover:bg-base-200">
									<td class="px-4 py-3">{formatDistanceLabel(distance)}</td>
									<td class="px-4 py-3 font-medium">{time}</td>
								</tr>
							{/each}
						</tbody>
					</table>
				</div>
			</div>

			<div class="mb-6">
				<h3 class="text-primary mb-3 text-lg font-bold">Training Paces (per km)</h3>
				<div class="bg-base-100 overflow-x-auto rounded-lg shadow-sm">
					<table class="table w-full">
						<thead>
							<tr>
								<th
									class="bg-base-200 px-4 py-3 text-left text-xs font-medium tracking-wider text-gray-500 uppercase"
									>Training Type</th
								>
								<th
									class="bg-base-200 px-4 py-3 text-left text-xs font-medium tracking-wider text-gray-500 uppercase"
									>Pace Range</th
								>
							</tr>
						</thead>
						<tbody>
							{#each Object.entries(calculatedPaces.training_paces) as [type, range]}
								<tr class="hover:bg-base-200">
									<td class="px-4 py-3">{formatTrainingType(type)}</td>
									<td class="px-4 py-3 font-medium">
										{#if 'min' in range && 'max' in range}
											{range.min} - {range.max}
										{:else if 'pace' in range}
											{range.pace}
										{/if}
									</td>
								</tr>
							{/each}
						</tbody>
					</table>
				</div>
			</div>
		</div>
	{/if}

	<div class="mt-6 text-sm text-gray-500">
		<p>
			The Hansons/Humphrey method uses race equivalency tables to calculate appropriate training
			paces for different types of workouts. This calculator is based on the methodology outlined in
			the Hansons Marathon Method book.
		</p>
	</div>
</div>
