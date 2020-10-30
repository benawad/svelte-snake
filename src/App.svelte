<script lang="ts">
  import { onMount } from "svelte/internal";

  let TICK_DELAY = 200;
  let GRID_SIZE = 20;
  let SNAKE_HEAD = 0;
  let lost = false;
  type Cell = "empty" | "snake" | "food";
  let grid: Cell[][] = [...Array(GRID_SIZE)].map(() =>
    [...Array(GRID_SIZE)].map(() => "empty")
  );
  let snakePosition: Array<[number, number]> = [[12, 13]];
  let direction = [0, 1];
  let gridWithSnake = grid;

  function getRandomInt(max) {
    return Math.floor(Math.random() * Math.floor(max));
  }

  function randomFood() {
    grid[getRandomInt(GRID_SIZE)][getRandomInt(GRID_SIZE)] = "food";
    grid[getRandomInt(GRID_SIZE)][getRandomInt(GRID_SIZE)] = "food";
  }

  randomFood();

  $: {
    for (let i = 0; i < gridWithSnake.length; i++) {
      for (let k = 0; k < gridWithSnake.length; k++) {
        if (gridWithSnake[i][k] === "snake") {
          gridWithSnake[i][k] = "empty";
        }
      }
    }
    snakePosition.forEach(([x, y]) => {
      gridWithSnake[x][y] = "snake";
    });
  }
  const fn = (n: number) => {
    setTimeout(() => {
      const [x, y] = snakePosition[SNAKE_HEAD];
      const [dx, dy] = direction;
      const newHead = [dx + x, y + dy] as [number, number];

      function isOutOfBounds(n: number) {
        return n < 0 || n > GRID_SIZE - 1;
      }

      if (isOutOfBounds(newHead[0]) || isOutOfBounds(newHead[1])) {
        lost = true;
        return;
      }

      let ateFood = false;
      if (gridWithSnake[newHead[0]][newHead[1]] === "food") {
        ateFood = true;
        randomFood();
      }

      const snakeBody = snakePosition.slice(
        0,
        snakePosition.length - (ateFood ? 0 : 1)
      );

      if (snakeBody.some((x) => x[0] === newHead[0] && x[1] === newHead[1])) {
        lost = true;
        return;
      }

      snakePosition = [newHead, ...snakeBody];
      fn(TICK_DELAY - Math.min(snakePosition.length, 15) * 10);
    }, n);
  };
  onMount(() => {
    fn(TICK_DELAY);
  });

  function restart() {
    snakePosition = [[12, 13]];
    direction = [0, 1];
    grid = [...Array(GRID_SIZE)].map(() =>
      [...Array(GRID_SIZE)].map(() => "empty")
    );
    gridWithSnake = grid;
    randomFood();
    lost = false;
    fn(TICK_DELAY);
  }
</script>

<style>
  .square {
    width: 20px;
    height: 20px;
    border: solid 1px #fff;
  }
  .empty {
    background-color: black;
  }
  .food {
    background-color: pink;
  }
  .snake {
    background-color: green;
  }
  .row {
    display: flex;
  }
  .center {
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .tcenter {
    text-align: center;
  }
  .restart {
    margin-top: 10px;
  }

  .enter {
    border: #000 1px solid;
    background-color: #eee;
    border-radius: 2px;
    padding: 5px;
  }
</style>

<svelte:window
  on:keydown={(e) => {
    switch (e.key) {
      case 'ArrowLeft':
        direction = [0, -1];
        break;
      case 'ArrowRight':
        direction = [0, 1];
        break;
      case 'ArrowUp':
        direction = [-1, 0];
        break;
      case 'ArrowDown':
        direction = [1, 0];
        break;
      case 'Enter':
        restart();
        break;
    }
  }} />

<main>
  {#if lost}
    <h1 class="tcenter">you lost</h1>
    <h3 class="tcenter">Hit <code class="enter">ENTER</code> to restart</h3>
  {/if}
  <h3 class="tcenter">snake length {snakePosition.length}</h3>
  <div class="center">
    <div>
      {#each gridWithSnake as row, i}
        <div class="row">
          {#each row as cell, k}
            <div
              on:click={() => (grid[i][k] = 'food')}
              class={`square ${cell}`} />
          {/each}
        </div>
      {/each}
    </div>
  </div>

  {#if lost}
    <div class="center restart">
      <button on:click={restart}> Start again </button>
    </div>
  {/if}
</main>
