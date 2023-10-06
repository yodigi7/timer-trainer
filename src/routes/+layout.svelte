<script lang="ts">
	import '../app.postcss';

	import TimerList from '$lib/timerlist.svelte';
	import type { ModalComponent } from '@skeletonlabs/skeleton';

	// Floating UI for Popups
	import { computePosition, autoUpdate, flip, shift, offset, arrow } from '@floating-ui/dom';
	import { Modal, storePopup } from '@skeletonlabs/skeleton';
	storePopup.set({ computePosition, autoUpdate, flip, shift, offset, arrow });

	import { initializeStores } from '@skeletonlabs/skeleton';
	initializeStores();

	const modalComponentRegistry: Record<string, ModalComponent> = {
		timerList: {
			ref: TimerList
		}
	};
</script>

<Modal components={modalComponentRegistry} />

<slot />
