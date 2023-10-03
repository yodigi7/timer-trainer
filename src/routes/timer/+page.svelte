<script lang="ts">
	let timers: any[] = [];
	let duration = 0;
	let note = '';

	let timeout: NodeJS.Timeout | undefined;

	function runTimersSequentially() {
		let currentIndex = 0;

		const runNextTimer = () => {
			if (currentIndex < timers.length) {
				const { duration, note } = timers[currentIndex];
				const callback = () => {
					let speech;
					if (!note.trim()) {
						speech = `Finished Timer ${currentIndex + 1}`;
					} else {
						speech = note;
					}
					speechSynthesis.speak(new SpeechSynthesisUtterance(speech));
				};
				return setTimeout(() => {
					callback();
					currentIndex++;
					timeout = runNextTimer(); // Start the next timer
				}, duration * 60 * 1000); // Convert minutes to seconds to milliseconds
			}
		};

		timeout = runNextTimer(); // Start the first timer
	}

	function addTimer() {
		timers.push({ duration, note });
		duration = 0;
		note = '';
		timers = timers;
	}

	function clearTimers() {
		stopTimers();
		timers = [];
		duration = 0;
		note = '';
	}

	function stopTimers() {
		if (timeout) {
			clearTimeout(timeout);
		}
		speechSynthesis.cancel();
	}
</script>

<div>
	<form>
		<label autofocus for="duration" class="label" id="duration-label">Duration (minutes):</label>
		<input type="number" id="duration" class="input" bind:value={duration} step="1" min="0" />

		<label class="label" for="note">Note:</label>
		<input type="text" class="input" id="note" bind:value={note} />

		<div class="flex gap-4 justify-center">
			<button type="submit" class="btn variant-filled" on:click={addTimer}>Add Timer</button>
			<button class="btn variant-filled" on:click={clearTimers}>Clear Timers</button>
			<button class="btn variant-filled" on:click={runTimersSequentially}>Start Timers</button>
			<button class="btn variant-filled" on:click={stopTimers}>Stop Timers</button>
		</div>
	</form>
	{#each timers as { duration, note }, index}
		<div>
			Timer: {duration}
			{#if note}
				- {note}
			{/if}
			<button
				on:click={() => {
					timers.splice(index, 1);
					timers = timers;
				}}
			>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					x="0px"
					y="0px"
					width="30"
					height="30"
					viewBox="0,0,256,256"
					transform="translate(0,7)"
					><g
						fill-opacity="0"
						fill="#dddddd"
						fill-rule="nonzero"
						stroke="none"
						stroke-width="1"
						stroke-linecap="butt"
						stroke-linejoin="miter"
						stroke-miterlimit="10"
						stroke-dasharray=""
						stroke-dashoffset="0"
						font-family="none"
						font-weight="none"
						font-size="none"
						text-anchor="none"
						style="mix-blend-mode: normal"><path d="M0,256v-256h256v256z" id="bgRectangle" /></g
					><g
						fill="#ffffff"
						fill-rule="nonzero"
						stroke="none"
						stroke-width="1"
						stroke-linecap="butt"
						stroke-linejoin="miter"
						stroke-miterlimit="10"
						stroke-dasharray=""
						stroke-dashoffset="0"
						font-family="none"
						font-weight="none"
						font-size="none"
						text-anchor="none"
						style="mix-blend-mode: normal"
						><g transform="scale(10.66667,10.66667)"
							><path
								d="M10,2l-1,1h-5v2h1v15c0,0.52222 0.19133,1.05461 0.56836,1.43164c0.37703,0.37703 0.90942,0.56836 1.43164,0.56836h10c0.52222,0 1.05461,-0.19133 1.43164,-0.56836c0.37703,-0.37703 0.56836,-0.90942 0.56836,-1.43164v-15h1v-2h-5l-1,-1zM7,5h10v15h-10zM9,7v11h2v-11zM13,7v11h2v-11z"
							/></g
						></g
					></svg
				>
			</button>
		</div>
	{/each}
</div>
