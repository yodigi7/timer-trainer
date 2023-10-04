<script>
	export let duration = 0;
	export let note = '';
</script>

Timer: {duration}
{#if note}
	- {note}
{/if}
