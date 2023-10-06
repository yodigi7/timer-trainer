<script lang="ts">
	import { ListBox, ListBoxItem, getModalStore } from '@skeletonlabs/skeleton';
    export let parent: any;
	const modalStore = getModalStore();
	const timers = $modalStore[0]?.meta?.timerNames;
    let selectedTimer = timers[0] ?? "";

    // Handle Form Submission
	function onFormSubmit(): void {
		if ($modalStore[0].response) $modalStore[0].response(selectedTimer);
		modalStore.close();
	}
</script>

<div class="modal-example-form card p-4 w-modal shadow-xl space-y-4">
	<header class="text-2xl font-bold">Select Timer</header>
    <article>Which timer would you like to choose?</article>
    <ListBox class="border border-surface-500 p-4 rounded-container-token">
        {#each timers as timer}
            <ListBoxItem bind:group={selectedTimer} name={timer} value={timer}>{timer}</ListBoxItem>
        {/each}
    </ListBox>
    <button class="btn {parent.buttonNeutral}" on:click={parent.onClose}>{parent.buttonTextCancel}</button>
    <button class="btn {parent.buttonPositive}" on:click={onFormSubmit}>Select</button>
</div>
