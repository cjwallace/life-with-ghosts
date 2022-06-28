<script lang="ts">
	// CONFIGURATION

	const GRIDSIZE = 40;
	const STEPTIME = 100; // ms
	const MARGIN = 2; // margin around whole svg
	const CELLSIZE = 5; // size of each cell
	const CELLPADDING = 2; // padding between cells
	const CELLSPACING = CELLSIZE + CELLPADDING;
	const COLORS = [
		'#00463E',
		'#328C93',
		'#459FA3',
		'#5CB0B2',
		'#73C0C0',
		'#8ACDCC',
		'#A1D9D6',
		'#B8E4E1',
		'whitesmoke'
	];
	const NGHOSTSTATES = COLORS.length - 1;

	// TYPES

	type Coords = {
		x: number;
		y: number;
	};

	type Cell = number;
	type Board = Cell[][];

	// LOGIC

	const modulo = (n: number, mod: number): number => {
		// regular % does not loop on negative numbers.
		// % is the remainder operator, not modulo.
		return ((n % mod) + mod) % mod;
	};

	const countNeighbours = (board: Board, coords: Coords): number => {
		const top = modulo(coords.y + 1, GRIDSIZE);
		const bottom = modulo(coords.y - 1, GRIDSIZE);
		const left = modulo(coords.x - 1, GRIDSIZE);
		const right = modulo(coords.x + 1, GRIDSIZE);
		const middleX = coords.x;
		const middleY = coords.y;
		// 0 means alive
		return (
			+(board[top][left] === 0) +
			+(board[top][middleX] === 0) +
			+(board[top][right] === 0) +
			+(board[middleY][left] === 0) +
			+(board[middleY][right] === 0) +
			+(board[bottom][left] === 0) +
			+(board[bottom][middleX] === 0) +
			+(board[bottom][right] === 0)
		);
	};

	const underpopulated = (nNeighbours: number) => nNeighbours < 2;
	const overpopulated = (nNeighbours: number) => nNeighbours > 3;
	const reproduction = (nNeighbours: number) => nNeighbours === 3;

	const rules = (board: Board, coords: Coords): Cell => {
		// state === 0 represents living cell
		// state > 0 is number of iterations cell has been dead, up to max
		const nNeighbours = countNeighbours(board, coords);

		let state = board[coords.y][coords.x];

		if (underpopulated(nNeighbours)) {
			state += 1;
		} else if (overpopulated(nNeighbours)) {
			state += 1;
		} else if (state !== 0 && reproduction(nNeighbours)) {
			state = 0;
		} else if (state !== 0) {
			state += 1;
		}

		if (state > NGHOSTSTATES) {
			state = NGHOSTSTATES;
		}

		return state;
	};

	const update = (board: Board): Board => {
		const initialBoard = board;
		const updatedBoard = board.map((row, rowIx) =>
			row.map((col, colIx) => {
				return rules(initialBoard, { x: colIx, y: rowIx });
			})
		);
		return updatedBoard;
	};

	// INITIALIZATION

	const initialize = (board: Board): Board => {
		return board.map((row) => row.map((cell) => (Math.random() < 0.2 ? 0 : NGHOSTSTATES)));
	};

	const emptyBoard: Board = new Array(GRIDSIZE).fill(0).map((_) => new Array(GRIDSIZE).fill(0));
	let board = initialize(emptyBoard);

	const step = () => {
		board = update(board);
	};

	let ticker = setInterval(() => step(), STEPTIME);
	let paused = false;

	// CONTROLS

	const reInitialize = () => {
		clearInterval(ticker);
		board = initialize(emptyBoard);
		ticker = setInterval(() => step(), STEPTIME);
		paused = false;
	};

	const pause = () => {
		paused = true;
		clearInterval(ticker);
	};

	const resume = () => {
		paused = false;
		ticker = setInterval(() => step(), STEPTIME);
	};
</script>

<div class="container">
	<button class="left" on:click={(e) => reInitialize()}>NEW</button>
	{#if !paused}
		<button class="right" on:click={(e) => pause()}>PAUSE</button>
	{:else}
		<button class="right" on:click={(e) => resume()}>RESUME</button>
	{/if}
	<svg
		viewBox="0 0 {GRIDSIZE * CELLSPACING + MARGIN} {GRIDSIZE * CELLSPACING + MARGIN}"
		width="100%"
	>
		{#each board as row, rowIx}
			{#each row as cell, colIx}
				<rect
					fill={COLORS[cell]}
					x={(colIx % GRIDSIZE) * CELLSPACING + MARGIN}
					y={(rowIx % GRIDSIZE) * CELLSPACING + MARGIN}
					width={CELLSIZE}
					height={CELLSIZE}
					rx="1"
				/>
			{/each}
		{/each}
	</svg>
</div>

<style>
	div.container {
		margin: auto;
		max-width: 28rem;
	}

	button {
		background-color: #1f2937;
		color: white;
		font: monospace;
		font-size: 0.875rem;
		padding: 0.25rem 0.5rem;
		display: inline;
		border-radius: 0.125rem;
	}

	button.left {
		float: left;
		margin-left: 0.125rem;
	}

	button.right {
		float: right;
		margin-right: 0.125rem;
	}

	button:focus {
		outline: 1px dotted;
		outline: 5px auto -webkit-focus-ring-color;
	}
</style>
