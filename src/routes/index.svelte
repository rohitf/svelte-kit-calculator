<script lang="ts">
	const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 0];
	const operations = {
		'+': (a) => (b) => a + b,
		'-': (a) => (b) => a - b,
		'×': (a) => (b) => a * b,
		'÷': (a) => (b) => a / b,
		xⁿ: (a) => (b) => a ** b
	} as const;

	type Operator = keyof typeof operations;
	const operators = Object.keys(operations) as Operator[];

	let displayResult = 0;
	let waitingForValue = false;
	let currentOp: Operator | null;
	let currentCalculation: Function | null;

	function clear() {
		displayResult = 0;
		currentOp = currentCalculation = null
		waitingForValue = true
	}

	function solve() {
		const result = currentCalculation(displayResult);
		clear();
		displayResult = result;
	}

	function numberPressed(n: number) {
		displayResult = waitingForValue ? n : n + displayResult * 10;
		waitingForValue &&= false
	}

	function applyArithmetic(op: Operator) {
		// If user hasn't submitted the previous calculation via "=", then solve that first
		if (currentCalculation && !waitingForValue) {
			solve()
		}
	
		currentCalculation = operations[op](displayResult);
		currentOp = op;
		waitingForValue = true;
	}

	// Support keyboard input
	function handleInput({ key }: KeyboardEvent) {
		if (!Number.isNaN(Number.parseInt(key))) {
			numberPressed(Number.parseInt(key));
		} else if (isOperator(key)) {
			applyArithmetic(key);
		} else if (key === 'Enter') {
			solve();
		} else if (key === 'Backspace' && !waitingForValue) {
			displayResult = Math.trunc(displayResult / 10);
		} else if (key === 'Escape') {
			clear();
		}
	}

	function isOperator(x): x is Operator {
		return operators.includes(x);
	}
</script>

<svelte:window on:keydown={handleInput} />
<main class="h-screen grid justify-center bg-gray-600">
	<div class="flex flex-col">
		<p class="mt-8 text-lg text-white text-center">use keyboard or buttons below</p>
		<!-- Calculator body -->
		<div class="flex flex-col h-fit bg-gray-200 bg-opacity-80 p-5 mt-6 border border-gray-400 rounded-md">
			<!-- Field where numbers and operations appear -->
			<div class="bg-white px-3 py-2">{displayResult}</div>
			<section class="flex mt-4 gap-4">
				<!-- Numbers buttons -->
				<div class="grid grid-rows-4 grid-cols-3 gap-4">
					{#each numbers as digit}
						<button
							class="px-6 text-xl bg-white hover:bg-blue-100"
							on:click={() => numberPressed(digit)}>{digit}</button
						>
					{/each}
				</div>
				<div>
					<!-- Operations buttons -->
					<div class="grid grid-rows-3 grid-cols-2 gap-4">
						{#each operators as op}
							<button
								class:bg-orange-500={op === currentOp}
								class="px-4 py-2 rounded-sm text-lg bg-orange-300 hover:bg-orange-500"
								on:click={() => applyArithmetic(op)}>{op}</button
							>
						{/each}
						<!-- Equals -->
						<button
							class="px-4 py-2 text-lg rounded-sm bg-blue-300 hover:bg-blue-400"
							on:click={solve}
						>
							=
						</button>
					</div>
					<!-- Clear -->
					<button
						class="w-full mt-4 px-4 py-2 text-lg border bg-slate-300"
						on:click={clear}
					>
						🙅‍♂️🙅‍♂️🙅‍♂️🙅‍♂️
					</button>
				</div>
			</section>
		</div>
	</div>
</main>

<style>
	main {
		background: linear-gradient(180deg, rgba(34, 193, 195, 1) 0%, rgba(253, 187, 45, 1) 100%);
	}
</style>
