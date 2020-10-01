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
  onMount(() => {
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
    fn(TICK_DELAY);
  });
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
</style>

<svelte:window
  on:keydown={(e) => {
    if (e.key === 'ArrowLeft') {
      direction = [0, -1];
    } else if (e.key === 'ArrowRight') {
      direction = [0, 1];
    } else if (e.key === 'ArrowUp') {
      direction = [-1, 0];
    } else if (e.key === 'ArrowDown') {
      direction = [1, 0];
    }
  }} />

<main>
  {#if lost}
    <h1 class="tcenter">you lost</h1>
  {/if}
  <h1 class="tcenter">snake length {snakePosition.length}</h1>
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
</main>
