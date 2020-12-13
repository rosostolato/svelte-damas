<script lang="ts">
  enum Player {
    Empty,
    Red,
    Black,
  }

  enum TurnState {
    SelectPiece,
    SelectMove,
  }

  let turn: Player
  let game: Player[]
  let state: TurnState
  let selectedPiece: number | null

  resetGame()

  function resetGame() {
    selectedPiece = null
    state = TurnState.SelectPiece
    turn = Player.Red

    game = Array.from(Array(8 * 8)).map((_, i) => {
      const row = Math.floor(i / 8)

      if (row < 3 && isGameCell(i)) {
        return Player.Black
      }

      if (row > 4 && isGameCell(i)) {
        return Player.Red
      }

      return Player.Empty
    })
  }

  function onSelect(i: number) {
    switch (state) {
      case TurnState.SelectPiece:
        if (game[i] === turn) {
          state = TurnState.SelectMove
          selectedPiece = i
        }
        break

      case TurnState.SelectMove:
        if (selectedPiece && isGameCell(i) && game[i] === Player.Empty) {
          game[i] = turn
          game[selectedPiece] = Player.Empty
          turn = turn === Player.Red ? Player.Black : Player.Red
          state = TurnState.SelectPiece
          selectedPiece = null
        }
        break
    }
  }

  function isGameCell(i: number) {
    const row = Math.floor(i / 8)
    const cell = i % 8
    return ((row % 2) + cell) % 2 === 1
  }
</script>

<style lang="scss">
  .table {
    width: 650px;
    height: 640px;
    display: grid;
    grid-template-rows: repeat(8, 1fr);
    grid-template-columns: repeat(8, 1fr);

    .cell {
      position: relative;

      @for $i from 1 through 8 {
        &.row-#{$i}:nth-child(2n + 1) {
          background-color: if($i % 2 == 0, #af8460, #e8d2af);
        }

        &.row-#{$i}:nth-child(2n) {
          background-color: if($i % 2 == 0, #e8d2af, #af8460);
        }
      }

      &.selected {
        border: 5px solid red;
      }

      &.cell-1,
      &.cell-2 {
        .piece {
          position: absolute;
          left: calc(50% - 20px);
          top: calc(50% - 20px);
          width: 40px;
          height: 40px;
          border-radius: 100%;
        }
      }

      &.cell-1 .piece {
        background-color: red;
      }

      &.cell-2 .piece {
        background-color: black;
      }
    }
  }
</style>

<div class="table">
  {#each game as cell, i}
    <div
      class={`cell cell-${cell} row-${Math.floor(i / 8) + 1}`}
      class:selected={selectedPiece === i}
      on:click={() => onSelect(i)}>
      <div class="piece" />
    </div>
  {/each}
</div>
