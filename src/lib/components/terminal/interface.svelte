<script lang="ts">
	import { onMount } from 'svelte';

	let termHistory: string[] = $state([]);
	let inputField: string = $state('');

	// buffer for user input
	let inputBuffer: string = '';

	let inputElement: HTMLDivElement;

	let prevCmds: string[] = [];
	let prevCmdIndex: number = 0;

	let container: HTMLElement; // window container
	let scrollable = $state(false);
	let isScrolledBot = $state(false);

	let inputClasses = $derived(!isScrolledBot ? 'border-zinc-800 border-2' : 'border-none');

	const handleKeyPress = (e: KeyboardEvent) => {
		const key = e.key;

		// Handle copy/paste/select all
		// Let browser handle ctrl commands naturally
		if (e.ctrlKey) {
			switch (key) {
				case 'c':
				case 'C':
					// Copy
					return;
				case 'v':
				case 'V':
					// Paste
					e.preventDefault();
					handlePaste();
					return;
				case 'a':
				case 'A':
					// Select all
					e.preventDefault();
					selectAllInput();
					return;
				case 'x':
				case 'X':
					// Cut
					e.preventDefault();
					handleCut();
					return;
			}
		}

		switch (key) {
			case 'Backspace':
				inputBuffer = inputBuffer.slice(0, -1);
				updateInputContent();
				e.preventDefault(); // Prevent default to avoid double handling
				break;
			case 'Enter':
				handleSend();
				e.preventDefault();
				break;
			case 'ArrowUp':
				console.log("A")
				break;
			case 'Tab':
				console.log("ADW")
				break;
			default:
				const isAlphanumeric = e.code.startsWith('Key') || e.code.startsWith('Digit') || e.code === 'Space';
				if (isAlphanumeric) {
					inputBuffer += key;
					updateInputContent();
					e.preventDefault(); // Prevent default to avoid double handling
				}
				break;
		}
	};

	const updateInputContent = () => {
		if (inputElement) {
			inputElement.textContent = inputBuffer;
			// Move cursor to end
			const range = document.createRange();
			const sel = window.getSelection();
			range.selectNodeContents(inputElement);
			range.collapse(false); // collapse to end
			sel?.removeAllRanges();
			sel?.addRange(range);
		}
	};

	const handlePaste = async () => {
		try {
			const clipboardText = await navigator.clipboard.readText();
			// Remove any newlines and add to buffer
			const cleanText = clipboardText.replace(/\r?\n/g, ' ');
			inputBuffer += cleanText;
			updateInputContent();
		} catch (err) {
			console.warn('Failed to read clipboard:', err);
		}
	};

	const selectAllInput = () => {
		if (inputElement && inputBuffer.length > 0) {
			const range = document.createRange();
			const sel = window.getSelection();
			range.selectNodeContents(inputElement);
			sel?.removeAllRanges();
			sel?.addRange(range);
		}
	};

	const handleCut = async () => {
		try {
			const sel = window.getSelection();
			if (sel && sel.toString().length > 0) {
				// Copy selected text to clipboard
				const selectedText = sel.toString();
				await navigator.clipboard.writeText(selectedText);
				
				const range = sel.getRangeAt(0);
				range.deleteContents();
				
				// Update our buffer to match the current content
				inputBuffer = inputElement.textContent || '';
			} else if (inputBuffer.length > 0) {
				// If nothing selected, cut all text
				await navigator.clipboard.writeText(inputBuffer);
				inputBuffer = '';
				updateInputContent();
			}
		} catch (err) {
			console.warn('Failed to cut text:', err);
		}
	};

	const handleSend = () => {
		if (inputBuffer === '') return;
		prevCmds.unshift(inputBuffer);
		termHistory = [...termHistory, '> ' + inputBuffer];
		inputField = '';
		inputBuffer = '';
		
		// Clear the input element
		if (inputElement) {
			inputElement.textContent = '';
		}

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

	const focusInput = () => {
		inputElement.focus();
	}

	onMount(() => checkScroll());
</script>

<div class="flex h-full flex-col font-mono text-white">
	<div bind:this={container} onscroll={checkScroll} class="overflow-auto bg-black pt-3 pl-3">
		<span class="">Type 'help' for a list of available commands.</span>
		{#each termHistory as cmd}
			<div class="">{cmd}</div>
		{/each}
	</div>

	<div class="inline-flex bg-black pl-3 ${inputClasses}">
		<label for="prompt" class="mr-2">
			<!-- Escaped '>' -->
			<span>&gt;</span>
		</label>

		<div
			id="prompt"
			class="w-full border-none bg-black p-0 pl-1 text-white caret-neutral-50 outline-none focus:ring-0 overflow-y-auto overflow-x-hidden break-words whitespace-pre-wrap"
			aria-multiline="true"
			role="textbox"
			tabindex="-1"
			contenteditable="true"
			bind:this={inputElement}
			onkeydown={handleKeyPress}
			oninput={() => {
				inputField = inputElement.textContent || '';
				inputBuffer = inputElement.textContent || '';
			}}
			placeholder="Type something and press Enter"
		></div>
	</div>
	<!-- Temporary (or not) thing so that it looks like the terminal takes up the whole screen -->
	<!-- It is a button instead of a div for accessibility -->
	<button class="flex-1 bg-black" onclick={focusInput} aria-label="Focus terminal input"></button>
</div>
