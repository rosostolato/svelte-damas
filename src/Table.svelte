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

  let selectedPiece: number | null
  let currPlayer: Player
  let squares: Player[]
  let turnState: TurnState
  let playerEats: Record<Player, number>

  resetGame()

  function resetGame() {
    selectedPiece = null
    turnState = TurnState.SelectPiece
    currPlayer = Player.Red
    playerEats = {
      [Player.Red]: 0,
      [Player.Black]: 0,
      [Player.Empty]: 0,
    }

    squares = Array(8 * 8)
      .fill(0)
      .map((_, i) => {
        const row = Math.floor(i / 8)

        if (row < 3 && isValidSquare(i)) {
          return Player.Black
        }

        if (row > 4 && isValidSquare(i)) {
          return Player.Red
        }

        return Player.Empty
      })
  }

  function onSelect(pos: number) {
    switch (turnState) {
      case TurnState.SelectPiece:
        1
        if (squares[pos] === currPlayer) {
          turnState = TurnState.SelectMove
          selectedPiece = pos
        }
        break

      case TurnState.SelectMove:
        if (!selectedPiece) {
          turnState = TurnState.SelectPiece
          return
        }

        if (validateMove(selectedPiece, pos)) {
          currPlayer = currPlayer === Player.Red ? Player.Black : Player.Red
          turnState = TurnState.SelectPiece
          selectedPiece = null
        }
        break
    }
  }

  function validateMove(sourcePos: number, destPos: number) {
    if (!isValidSquare(destPos) || squares[destPos] !== Player.Empty) {
      return false
    }

    const diff = destPos - sourcePos
    const move = isRounded(diff / 7) ? 7 : 9
    const distance = diff / move
    const signal = distance / Math.abs(distance)
    const dir = signal * move

    if (
      !isRounded(distance) ||
      Math.abs(distance) > 2 ||
      (distance > 0 && currPlayer === Player.Red) ||
      (distance < 0 && currPlayer === Player.Black)
    ) {
      return false
    }

    // Eat move
    if (Math.abs(distance) === 2) {
      const otherPlayer = currPlayer === Player.Red ? Player.Black : Player.Red

      if (squares[sourcePos + dir] !== otherPlayer) {
        return false
      }

      squares[sourcePos + dir] = Player.Empty
      playerEats[currPlayer]++
    }

    squares[destPos] = currPlayer
    squares[selectedPiece!] = Player.Empty
    return true
  }

  function isValidSquare(pos: number) {
    const row = Math.floor(pos / 8)
    const cell = pos % 8
    return ((row % 2) + cell) % 2 === 1
  }

  function isRounded(num: number) {
    return Math.round(num) / num === 1
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

  .info {
    margin-top: 20px;

    span {
      margin-left: 10px;
    }
  }
</style>

<div class="table">
  {#each squares as cell, i}
    <div
      class={`cell cell-${cell} row-${Math.floor(i / 8) + 1}`}
      class:selected={selectedPiece === i}
      on:click={() => onSelect(i)}>
      <div class="piece" />
    </div>
  {/each}
</div>

<div class="info">
  <button on:click={() => resetGame()}>Reset Game</button>
  <span>Player Red: {playerEats[Player.Red]}</span>
  <span>Player Black: {playerEats[Player.Black]}</span>
</div>
