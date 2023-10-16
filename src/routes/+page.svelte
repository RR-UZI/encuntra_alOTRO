<script lang="ts">
    import { emoji } from "./emoji";

    type State = 'start' | 'playing' | 'paused' | 'won' | 'lost'

    let state: State = 'start'
    let size = 20
    let grid = createGrid()
    let maxMatches = grid.length / 2
    let selected: number[] = []
    let matches: string[] = []
    let timerId: number | null = null
	let time = 60

    function startGameTimer() {
		function countdown() {
			state !== 'paused' && (time -= 1)
		}
		timerId = setInterval(countdown, 1000)
	}

    function gameLost() {
		state = 'lost'
        resetGame()

	}

    function createGrid() {
        let cards = new Set<string>()
        let maxSize = size / 2

        while (cards.size < maxSize) {
            const randomIndex = Math.floor(Math.random() * emoji.length)
            cards.add(emoji[randomIndex])
        }

        return shuffle([...cards, ...cards])
    }

    function shuffle<Items>(array: Items[]) {
        return array.sort(() => Math.random() - 0.5)
    }

    function selectCard(cardIndex: number) {
		selected = selected.concat(cardIndex)
	}

    function matchCards() {
		// array destructuring can have any name for the values
		const [first, second] = selected

		if (grid[first] === grid[second]) {
			matches = matches.concat(grid[first])
		}

		// clear selected
		setTimeout(() => selected = [], 300)
	}

    function gameWon() {
		state = 'won'
        resetGame()

	}
    
    function resetGame() {
		timerId && clearInterval(timerId)
		grid = createGrid()
		maxMatches = grid.length / 2
		selected = []
		matches = []
		timerId = null
		time = 60
	}

    function pauseGame(e: KeyboardEvent) {
		if (e.key === 'Escape') {
			switch (state) {
				case 'playing':
					state = 'paused'
					break
				case 'paused':
					state = 'playing'
					break
			}
		}
	}

    
    $: selected.length === 2 && matchCards()
	$: maxMatches === matches.length && gameWon()
    $: if (state === 'playing') {
		//	in case you pause the game
		!timerId && startGameTimer()
	}
    $: time === 0 && gameLost()

</script>

<svelte:window on:keydown={pauseGame} />

{#if state === 'paused'}
	<h1>Juego Pausado</h1>
{/if}

{#if state === 'start'}
	<h1>Encuentra al OTRO</h1>
	<button on:click={() => state = 'playing'}>
    Play
  </button>
{/if}

{#if state === 'playing'}
    <h1 class="timer" class:pulse={time <= 10}>
		{time}
	</h1>
    <div class="matches">
		{#each matches as match}
			<div>{match}</div>
		{/each}
	</div>
	<div class="cards">
		{#each grid as card, cardIndex}
			{@const isSelected = selected.includes(cardIndex)}
			{@const isSelectedOrMatch = selected.includes(cardIndex) || matches.includes(card)}
			{@const match = matches.includes(card)}

			<button
				class="card"
				class:selected={isSelected}
				disabled={isSelectedOrMatch}
                class:flip={isSelectedOrMatch}
				on:click={() => selectCard(cardIndex)}
			>
				<div class="back" class:match>{card}</div>
			</button>
		{/each}
	</div>
{/if}

{#if state === 'lost'}
	<h1>Perdisteee! Desinstala manco 💩</h1>
	<button on:click={() => state = 'playing'}>
    Jugar de Nuevo
  </button>
{/if}

{#if state === 'won'}
	<h1>GG EZ 🥱</h1>
	<button on:click={() => state = 'playing'}>
    Jugar de Nuevo
  </button>
{/if}

<style>
	.cards {
		display: grid;
		grid-template-columns: repeat(5, 1fr);
		gap: 0.4rem;
	}

	.card {
		height: 140px;
		width: 140px;
		font-size: 4rem;
		background-color: var(--bg-2);
        transition: rotate 0.3s ease-out;
		transform-style: preserve-3d;

		&.selected {
			border: 4px solid var(--border);
		}

        &.flip {
			rotate: y 180deg;
			pointer-events: none;
		}

        & .back {
			position: absolute;
			inset: 0;
			display: grid;
			place-content: center;
			backface-visibility: hidden;
			rotate: y 180deg;
		}

        
        & .match {
			transition: opacity 0.3s ease-out;
			opacity: 0.4;
		}

	}
    .matches {
		display: flex;
		gap: 1rem;
		margin-block: 2rem;
		font-size: 3rem;
	}
    .timer {
		transition: color 0.3s ease;
	}

	.pulse {
		color: var(--pulse);
		animation: pulse 1s infinite ease;
	}

	@keyframes pulse {
		to {
			scale: 1.4;
		}
	}
</style>

