<script lang="ts">
	import { onMount } from 'svelte';

	let cmdHistory: string[] = $state([]);
	let field: string = $state('');

	let userTyped: string = '';

	let prevCmds: string[] = [];
	let prevCmdIndex: number = 0;

	let container: HTMLElement; // window container
	let scrollable = $state(false);
	let isScrolledBot = $state(false);

	let inputClasses = $derived(!isScrolledBot ? 'border-zinc-800 border-2' : 'border-none');

	const handleKeyDown = (e: KeyboardEvent) => {
		const key = e.key;

		switch (key) {
			case 'Backspace':
				userTyped = userTyped.slice(0, -1);
				break;
			case 'Enter':
				handleSend();
				break;

			default:
				const isAlphanumeric = e.code.startsWith('Key') || e.code.startsWith('Digit');
				if (isAlphanumeric) userTyped += key;

				break;
		}
	};

	const handleSend = () => {
		if (userTyped === '') return;
		cmdHistory = [...cmdHistory, '> ' + userTyped];
		field = '';
		userTyped = '';

		// Auto-scroll to bottom after adding command
		setTimeout(() => {
			if (container) {
				container.scrollTop = container.scrollHeight;
			}
		}, 0);
	};

	const checkScroll = () => {
		if (!container) return;
		const { scrollTop, clientHeight, scrollHeight } = container;
		scrollable = scrollHeight > clientHeight;
		isScrolledBot = scrollTop + clientHeight >= scrollHeight;
	};

	onMount(() => checkScroll());
</script>

<div class="flex h-full flex-col font-mono text-white">
	<div bind:this={container} onscroll={checkScroll} class="overflow-auto bg-black pt-3 pl-3">
		<span class="">Type 'help' for a list of available commands.</span>
		{#each cmdHistory as cmd}
			<div class="">{cmd}</div>
		{/each}
	</div>

	<div class="inline-flex bg-black pl-3 ${inputClasses}">
		<label for="prompt" class="mr-2">
			<!-- Escaped '>' -->
			<span>&gt;</span>
		</label>

		<input
			id="prompt"
			type="text"
			class="w-full border-none bg-black p-0 text-white caret-neutral-50 outline-none focus:ring-0"
			bind:value={field}
			onkeydown={handleKeyDown}
			placeholder="Type something and press Enter"
		/>
	</div>
	<!-- Temporary (or not) thing so that it looks like the terminal takes up the whole screen -->
	<div class="flex-1 bg-black"></div>
</div>
