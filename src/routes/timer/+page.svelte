<script lang="ts">
	import Timer from '$lib/timer.svelte';
	import { getModalStore, type ModalSettings } from '@skeletonlabs/skeleton';
	const modalStore = getModalStore();

	let timers: any[] = [];

	let timeout: NodeJS.Timeout | undefined;
	let playing = false;

	async function loadTimer() {
		let timerNames = JSON.parse(localStorage.getItem('timerTrainer-timerNames') || '[]');
		// No timers saved
		if (timerNames.length === 0) {
			const modal: ModalSettings = {
				type: 'alert',
				title: 'No timers found!',
				body: 'No timers found!'
			};
			modalStore.trigger(modal);
			return;
		}
		// Get timer list to display
		new Promise<string | false>((resolve) => {
			const modal: ModalSettings = {
				type: 'component',
				component: 'timerList',
				meta: { timerNames: timerNames },
				response: (r: string | false) => {
					resolve(r);
				}
			};
			modalStore.trigger(modal);
		}).then((timerName) => {
			console.log(timerName);
			const timersString = localStorage.getItem(`timerTrainer-${timerName}`);
			console.log(timersString);
			clearTimers();
			timers = JSON.parse(timersString || '[]');
		})
	}

	// Get duration and note from modal
	async function addTimer() {
		// Open modal to prompt for duration
		new Promise<number | false>((resolve) => {
			const modal: ModalSettings = {
				type: 'prompt',
				// Data
				title: 'Duration',
				body: 'How long should this last?',
				value: 0,
				valueAttr: { type: 'number', required: true },
				response: (r: number | false) => {
					resolve(r);
				}
			};
			modalStore.trigger(modal);
		}).then((duration) => {
			if (duration || duration === 0) {
				console.log('after duration');
				new Promise<string | false>((resolve) => {
					console.log('after duration');
					const modal: ModalSettings = {
						type: 'prompt',
						// Data
						title: 'Duration',
						body: 'How long should this last?',
						value: '',
						valueAttr: { type: 'string', required: false },
						response: (r: string | false) => {
							resolve(r);
						}
					};
					modalStore.trigger(modal);
				}).then((note) => {
					console.log('after note');
					if (note) {
						note = note.trim();
						_addTimer(duration, note);
					} else {
						_addTimer(duration, '');
					}
				});
			}
		});
	}

	async function saveTimers() {
		// Open modal to prompt for name
		new Promise<string | false>((resolve) => {
			const modal: ModalSettings = {
				type: 'prompt',
				// Data
				title: 'Enter Timer Name',
				body: 'What would you like to name this?',
				value: '',
				valueAttr: { type: 'text', required: true },
				response: (r: string | false) => {
					resolve(r);
				}
			};
			modalStore.trigger(modal);
		}).then((name) => {
			if (name) {
				_saveTimer(name);
			}
		});
	}

	function _saveTimer(name: string) {
		let timerNames = JSON.parse(localStorage.getItem('timerTrainer-timerNames') || '["default"]');
		timerNames.push(name);
		localStorage.setItem('timerTrainer-timerNames', JSON.stringify(timerNames));
		localStorage.setItem(`timerTrainer-${name}`, JSON.stringify(timers));
		localStorage.setItem(`timerTrainer-default`, JSON.stringify(timers));
	}

	function deleteTimers() {
		// TODO: delete from one of many options
		let timerNames: string[] = JSON.parse(localStorage.getItem('timerTrainer-timerNames') || '[]');
		timerNames.forEach((name) => {
			localStorage.removeItem(`timerTrainer-${name}`);
		});
		localStorage.removeItem('timerTrainer-timerNames');
		localStorage.removeItem('timerTrainer-default');
	}

	function runTimersSequentially() {
		playing = true;
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
					// Last one so update that no long playing
					if (currentIndex == timers.length-1) {
						playing = false;
					}
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

	function _addTimer(duration: number, note: string) {
		timers.push({ duration, note });
		duration = 0;
		note = '';
		timers = timers;
	}

	function clearTimers() {
		stopTimers();
		timers = [];
	}

	function stopTimers() {
		if (timeout) {
			clearTimeout(timeout);
		}
		speechSynthesis.cancel();
		playing = false;
	}
</script>

<div class="card m-auto pt-6 h-screen w-11/12">
	<h1 class="h1 text-center pb-8">Timer Trainer</h1>
	<div class="flex gap-4 justify-center pb-8">
		<button type="submit" class="btn variant-filled" on:click={addTimer}>Add Timer</button>
	</div>
	<div class="max-h-96 overflow-y-scroll pl-28">
		{#each timers as { duration, note }, index}
			<div class="block">
				<Timer {duration} {note} />
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
			<br/>
		{/each}
	</div>
	<div class="flex justify-end gap-3 flex-wrap py-8 pr-20">
		{#if timers.length}
			{#if playing}
				<button class="btn variant-filled" on:click={stopTimers}>Stop Timers</button>
				{:else}
				<button class="btn variant-filled" on:click={runTimersSequentially}>Start Timers</button>
			{/if}
		{/if}
	</div>
	<div class="flex justify-end gap-3 flex-wrap pr-20">
		{#if timers.length}
			<button class="btn variant-filled" on:click={saveTimers}>Save Timers</button>
			<button class="btn variant-filled" on:click={clearTimers}>Clear Timers</button>
		{/if}
		<button class="btn variant-filled" on:click={loadTimer}>Load Timer</button>
		<button class="btn variant-filled" on:click={deleteTimers}>Delete All Timers</button>
	</div>
</div>
