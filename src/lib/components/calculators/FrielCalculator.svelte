<script>
	import { frielCalculate, timeToSeconds, formatPace } from '$lib/calculatorUtils';

	// Form data with reactive state
	let formData = $state({
		ltPaceMinutes: '',
		ltPaceSeconds: '',
		lactateThresholdHr: '',
		functionalThresholdPower: ''
	});

	// Results storage
	let zonesResult = $state(null);
	let loading = $state(false);
	let error = $state(null);

	// Calculate training zones based on inputs
	function calculateZones() {
		loading = true;
		error = null;

		try {
			// At least one of LT pace, LTHR, or FTP must be provided
			if (
				formData.ltPaceMinutes === '' &&
				formData.ltPaceSeconds === '' &&
				formData.lactateThresholdHr === '' &&
				formData.functionalThresholdPower === ''
			) {
				throw new Error('Please provide at least one of the threshold values');
			}

			// Process LT pace input
			let ltPaceSeconds = null;
			if (formData.ltPaceMinutes !== '' || formData.ltPaceSeconds !== '') {
				const minutes = formData.ltPaceMinutes ? parseInt(formData.ltPaceMinutes, 10) : 0;
				const seconds = formData.ltPaceSeconds ? parseInt(formData.ltPaceSeconds, 10) : 0;

				if (minutes === 0 && seconds === 0) {
					throw new Error('Lactate threshold pace cannot be zero');
				}

				ltPaceSeconds = minutes * 60 + seconds;
			}

			// Process LTHR input
			let lthr = null;
			if (formData.lactateThresholdHr !== '') {
				lthr = parseInt(formData.lactateThresholdHr, 10);
				if (isNaN(lthr) || lthr < 100 || lthr > 220) {
					throw new Error('Please enter a valid LTHR between 100 and 220');
				}
			}

			// Process FTP input
			let ftp = null;
			if (formData.functionalThresholdPower !== '') {
				ftp = parseInt(formData.functionalThresholdPower, 10);
				if (isNaN(ftp) || ftp < 50) {
					throw new Error('Please enter a valid FTP (minimum 50 watts)');
				}
			}

			// Calculate zones using the calculatorUtils function
			zonesResult = frielCalculate(lthr, ftp, ltPaceSeconds);
		} catch (err) {
			error = err.message;
			zonesResult = null;
		} finally {
			loading = false;
		}
	}
</script>

<div class="rounded-lg bg-white p-6 shadow-md">
	<h1 class="mb-4 text-xl font-bold">Friel Training Zones Calculator</h1>
	<p class="mb-4 text-gray-600">
		Calculate training zones based on Joe Friel's methodology by entering at least one threshold
		value.
	</p>

	<div class="grid gap-6 md:grid-cols-2">
		<!-- Left column: Input form -->
		<div>
			<div class="mb-4">
				<label for="lactateThresholdHr" class="mb-2 block text-gray-700"
					>Lactate Threshold Heart Rate (LTHR)</label
				>
				<input
					type="number"
					id="lactateThresholdHr"
					bind:value={formData.lactateThresholdHr}
					min="100"
					max="220"
					placeholder="Enter LTHR in bpm"
					class="w-full rounded border p-2"
				/>
			</div>

			<div class="mb-4">
				<label for="functionalThresholdPower" class="mb-2 block text-gray-700"
					>Functional Threshold Power (FTP)</label
				>
				<input
					type="number"
					id="functionalThresholdPower"
					bind:value={formData.functionalThresholdPower}
					min="50"
					placeholder="Enter FTP in watts"
					class="w-full rounded border p-2"
				/>
			</div>

			<div class="mb-4">
				<label class="mb-2 block text-gray-700">Lactate Threshold Pace (per km)</label>
				<div class="grid grid-cols-2 gap-4">
					<div>
						<label class="block text-sm text-gray-600">Minutes</label>
						<input
							type="number"
							bind:value={formData.ltPaceMinutes}
							min="0"
							placeholder="MM"
							class="w-full rounded border p-2"
						/>
					</div>
					<div>
						<label class="block text-sm text-gray-600">Seconds</label>
						<input
							type="number"
							bind:value={formData.ltPaceSeconds}
							min="0"
							max="59"
							placeholder="SS"
							class="w-full rounded border p-2"
						/>
					</div>
				</div>
			</div>

			<button
				class="rounded bg-blue-500 px-4 py-2 font-bold text-white hover:bg-blue-600 disabled:bg-gray-400"
				on:click={calculateZones}
				disabled={loading}
			>
				{loading ? 'Calculating...' : 'Calculate Zones'}
			</button>

			{#if error}
				<div class="my-4 border-l-4 border-red-500 bg-red-100 p-4 text-red-700" role="alert">
					<p>{error}</p>
				</div>
			{/if}
		</div>

		<!-- Right column: Results -->
		<div>
			{#if zonesResult}
				<div>
					<!-- Heart Rate Zones -->
					{#if zonesResult.hr_zones}
						<div class="mb-6">
							<h2 class="text-lg font-bold">Heart Rate Zones (LTHR: {zonesResult.lthr} bpm)</h2>
							<div class="mt-2 overflow-x-auto">
								<table class="min-w-full border bg-white">
									<thead>
										<tr>
											<th class="border-b border-gray-200 bg-gray-50 px-4 py-2 text-left">Zone</th>
											<th class="border-b border-gray-200 bg-gray-50 px-4 py-2 text-left"
												>Range (bpm)</th
											>
											<th class="border-b border-gray-200 bg-gray-50 px-4 py-2 text-left"
												>Description</th
											>
										</tr>
									</thead>
									<tbody>
										{#each Object.entries(zonesResult.hr_zones) as [zone, data]}
											<tr>
												<td class="border-b border-gray-200 px-4 py-2"
													>{zone.replace('zone', 'Zone ')}</td
												>
												<td class="border-b border-gray-200 px-4 py-2">{data.min} - {data.max}</td>
												<td class="border-b border-gray-200 px-4 py-2">{data.description}</td>
											</tr>
										{/each}
									</tbody>
								</table>
							</div>
						</div>
					{/if}

					<!-- Power Zones -->
					{#if zonesResult.power_zones}
						<div class="mb-6">
							<h2 class="text-lg font-bold">Power Zones (FTP: {zonesResult.ftp} watts)</h2>
							<div class="mt-2 overflow-x-auto">
								<table class="min-w-full border bg-white">
									<thead>
										<tr>
											<th class="border-b border-gray-200 bg-gray-50 px-4 py-2 text-left">Zone</th>
											<th class="border-b border-gray-200 bg-gray-50 px-4 py-2 text-left"
												>Range (watts)</th
											>
											<th class="border-b border-gray-200 bg-gray-50 px-4 py-2 text-left"
												>Description</th
											>
										</tr>
									</thead>
									<tbody>
										{#each Object.entries(zonesResult.power_zones) as [zone, data]}
											<tr>
												<td class="border-b border-gray-200 px-4 py-2"
													>{zone.replace('zone', 'Zone ')}</td
												>
												<td class="border-b border-gray-200 px-4 py-2">
													{#if data.min && data.max}
														{data.min} - {data.max}
													{:else if data.note}
														{data.note}
													{/if}
												</td>
												<td class="border-b border-gray-200 px-4 py-2">{data.description}</td>
											</tr>
										{/each}
									</tbody>
								</table>
							</div>
						</div>
					{/if}

					<!-- Pace Zones -->
					{#if zonesResult.pace_zones}
						<div class="mb-6">
							<h2 class="text-lg font-bold">Pace Zones (LT pace: {zonesResult.lt_pace} min/km)</h2>
							<div class="mt-2 overflow-x-auto">
								<table class="min-w-full border bg-white">
									<thead>
										<tr>
											<th class="border-b border-gray-200 bg-gray-50 px-4 py-2 text-left">Zone</th>
											<th class="border-b border-gray-200 bg-gray-50 px-4 py-2 text-left"
												>Range (min/km)</th
											>
											<th class="border-b border-gray-200 bg-gray-50 px-4 py-2 text-left"
												>Description</th
											>
										</tr>
									</thead>
									<tbody>
										{#each Object.entries(zonesResult.pace_zones) as [zone, data]}
											<tr>
												<td class="border-b border-gray-200 px-4 py-2"
													>{zone.replace('zone', 'Zone ')}</td
												>
												<td class="border-b border-gray-200 px-4 py-2">
													{#if data.min && data.max}
														{data.min} - {data.max}
													{:else if data.note}
														{data.note}
													{/if}
												</td>
												<td class="border-b border-gray-200 px-4 py-2">{data.description}</td>
											</tr>
										{/each}
									</tbody>
								</table>
							</div>
						</div>
					{/if}
				</div>
			{:else}
				<div class="rounded-lg bg-gray-100 p-6 text-center text-gray-500">
					<p>Enter at least one threshold value to calculate your training zones.</p>
				</div>
			{/if}
		</div>
	</div>
</div>
