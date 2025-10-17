<script lang="ts">
  // Game state (Svelte 5 reactivity with $state)
  let board = $state<(null | 'X' | 'O')[]>(Array(9).fill(null));
  let currentPlayer = $state<'X' | 'O'>('X');
  let winner = $state<null | 'X' | 'O'>(null);
  let isDraw = $state(false);
  let winningLine = $state<number[] | null>(null);

  const winningCombos: number[][] = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6]
  ];

  // PUBLIC_INTERFACE
  function handleCellClick(index: number) {
    /** Handle a click on the specified cell index. If valid, place current player's mark and evaluate game state. */
    if (board[index] || winner) return;
    // mutate through $state array in place, then reassign to trigger updates
    board[index] = currentPlayer;
    board = [...board];
    evaluateBoard();
    if (!winner && !isDraw) {
      currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
    }
  }

  function evaluateBoard() {
    // Check for win
    for (const combo of winningCombos) {
      const [a, b, c] = combo;
      if (board[a] && board[a] === board[b] && board[a] === board[c]) {
        winner = board[a];
        winningLine = combo;
        isDraw = false;
        return;
      }
    }
    // Check for draw
    if (board.every((c) => c !== null)) {
      isDraw = true;
      winner = null;
      winningLine = null;
    }
  }

  // PUBLIC_INTERFACE
  function resetGame() {
    /** Reset the game to initial state. */
    board = Array(9).fill(null);
    currentPlayer = 'X';
    winner = null;
    isDraw = false;
    winningLine = null;
  }

  // Accessibility: keyboard support
  function onCellKeydown(e: KeyboardEvent, index: number) {
    const key = e.key;
    if (key === 'Enter' || key === ' ') {
      e.preventDefault();
      handleCellClick(index);
    }
  }

  $effect(() => {
    // Keep document title updated with status
    const base = 'Tic-Tac-Toe';
    if (winner) document.title = `${base} – Winner: ${winner}`;
    else if (isDraw) document.title = `${base} – Draw`;
    else document.title = `${base} – ${currentPlayer}'s turn`;
  });
</script>

<svelte:head>
  <title>Tic-Tac-Toe</title>
  <meta name="description" content="Play a modern Tic-Tac-Toe game built with Svelte." />
</svelte:head>

<div class="page">
  <div class="card">
    <header class="header">
      <h1 class="title">Tic‑Tac‑Toe</h1>
      <p class="subtitle">
        {#if winner}
          <span class="status win">Winner: {winner}</span>
        {:else if isDraw}
          <span class="status draw">It's a draw!</span>
        {:else}
          <span class="status turn">Current turn: <strong>{currentPlayer}</strong></span>
        {/if}
      </p>
    </header>

    <div class="board" role="group" aria-label="Tic-Tac-Toe board">
      {#each board as cell, i (i)}
        <button
          class="cell {winningLine && winningLine.includes(i) ? 'win-highlight' : ''}"
          aria-label={cell ? `Cell ${i + 1}, ${cell}` : `Cell ${i + 1}, empty`}
          aria-disabled={!!(cell || winner)}
          disabled={!!(cell || winner)}
          onclick={() => handleCellClick(i)}
          onkeydown={(e) => onCellKeydown(e, i)}
          tabindex="0"
        >
          {#if cell}
            <span class="mark {cell === 'X' ? 'x' : 'o'}">{cell}</span>
          {/if}
        </button>
      {/each}
    </div>

    <footer class="footer">
      <button class="reset" onclick={resetGame} aria-label="Start a new game">New Game</button>
    </footer>
  </div>
</div>

<style>
  :root {
    --primary: #2563EB;
    --secondary: #F59E0B; /* also used for success */
    --error: #EF4444;
    --bg: #f9fafb;
    --surface: #ffffff;
    --text: #111827;

    --shadow-sm: 0 1px 2px rgba(0,0,0,0.05);
    --shadow-md: 0 4px 12px rgba(0,0,0,0.08);
    --radius: 16px;
    --radius-sm: 12px;
    --cell-size: clamp(80px, 10vw, 120px);
  }

  .page {
    width: 100%;
    min-height: 100vh;
    display: grid;
    place-items: center;
    background:
      radial-gradient(1200px 600px at 20% -10%, rgba(37,99,235,0.08), rgba(243,244,246,0) 60%),
      linear-gradient(180deg, rgba(37,99,235,0.10), rgba(249,250,251,0)) no-repeat;
    background-color: var(--bg);
    color: var(--text);
    padding: 24px;
  }

  .card {
    background: var(--surface);
    border-radius: var(--radius);
    box-shadow: var(--shadow-md);
    width: min(98vw, 520px);
    padding: 24px 20px 20px;
    border: 1px solid rgba(17,24,39,0.06);
    backdrop-filter: saturate(1.1);
  }

  .header {
    text-align: center;
    margin-bottom: 18px;
  }

  .title {
    margin: 0 0 6px 0;
    font-size: clamp(1.25rem, 2.5vw, 1.6rem);
    font-weight: 800;
    letter-spacing: 0.2px;
    background: linear-gradient(135deg, var(--primary), #60a5fa);
    -webkit-background-clip: text;
    background-clip: text;
    color: transparent;
  }

  .subtitle {
    margin: 0;
    font-size: 0.98rem;
    color: #374151;
  }

  .status {
    display: inline-block;
    padding: 6px 10px;
    border-radius: 999px;
    font-weight: 600;
    letter-spacing: 0.2px;
    box-shadow: var(--shadow-sm);
    background: #f3f4f6;
  }
  .status.turn strong {
    color: var(--primary);
  }
  .status.win {
    color: #065f46;
    background: linear-gradient(180deg, rgba(245,158,11,0.18), rgba(255,255,255,0.9));
    border: 1px solid rgba(245,158,11,0.35);
  }
  .status.draw {
    color: #b45309;
    background: linear-gradient(180deg, rgba(245,158,11,0.12), rgba(255,255,255,0.9));
    border: 1px solid rgba(245,158,11,0.25);
  }

  .board {
    display: grid;
    grid-template-columns: repeat(3, var(--cell-size));
    grid-template-rows: repeat(3, var(--cell-size));
    gap: 10px;
    justify-content: center;
    padding: 10px;
    border-radius: var(--radius-sm);
    background: linear-gradient(180deg, rgba(37,99,235,0.08), rgba(249,250,251,1));
    box-shadow: inset 0 1px 0 rgba(255,255,255,0.6), 0 1px 0 rgba(17,24,39,0.03);
    margin: 8px auto 14px;
  }

  .cell {
    width: var(--cell-size);
    height: var(--cell-size);
    border-radius: 14px;
    border: 1px solid rgba(17,24,39,0.08);
    background: linear-gradient(180deg, #ffffff, #f9fafb);
    box-shadow: var(--shadow-sm);
    display: grid;
    place-items: center;
    cursor: pointer;
    transition: transform 120ms ease, box-shadow 120ms ease, border-color 120ms ease, background 200ms ease;
    position: relative;
    outline: none;
  }
  .cell:hover:not([disabled]) {
    transform: translateY(-1px);
    box-shadow: 0 6px 14px rgba(0,0,0,0.08);
    border-color: rgba(37,99,235,0.35);
    background: linear-gradient(180deg, #ffffff, #eef2ff);
  }
  .cell:focus-visible {
    border-color: var(--primary);
    box-shadow: 0 0 0 3px rgba(37,99,235,0.25);
  }
  .cell[disabled] {
    cursor: default;
    opacity: 1;
  }

  .mark {
    font-size: clamp(2rem, 6vw, 3rem);
    font-weight: 800;
    line-height: 1;
    transition: transform 160ms ease, color 160ms ease, text-shadow 160ms ease;
    text-shadow: 0 2px 0 rgba(0,0,0,0.04);
  }
  .mark.x {
    color: var(--primary);
    text-shadow: 0 6px 18px rgba(37,99,235,0.25);
  }
  .mark.o {
    color: var(--error);
    text-shadow: 0 6px 18px rgba(239,68,68,0.25);
  }

  .win-highlight {
    background: linear-gradient(180deg, rgba(245,158,11,0.15), #fff);
    border-color: rgba(245,158,11,0.55);
    box-shadow: 0 8px 22px rgba(245,158,11,0.25);
  }

  .footer {
    display: flex;
    justify-content: center;
    margin-top: 4px;
  }

  .reset {
    appearance: none;
    border: 1px solid rgba(17,24,39,0.1);
    background: linear-gradient(180deg, #ffffff, #f9fafb);
    color: var(--text);
    padding: 10px 14px;
    font-weight: 700;
    border-radius: 12px;
    cursor: pointer;
    transition: transform 120ms ease, box-shadow 120ms ease, background 180ms ease, border-color 120ms ease;
    box-shadow: var(--shadow-sm);
  }
  .reset:hover {
    transform: translateY(-1px);
    box-shadow: 0 8px 18px rgba(0,0,0,0.08);
    background: linear-gradient(180deg, #ffffff, #eef2ff);
    border-color: rgba(37,99,235,0.35);
  }
  .reset:focus-visible {
    outline: none;
    box-shadow: 0 0 0 3px rgba(37,99,235,0.25);
    border-color: var(--primary);
  }

  @media (max-width: 380px) {
    :root { --cell-size: 82px; }
    .card { padding: 18px 14px 16px; }
  }
</style>
