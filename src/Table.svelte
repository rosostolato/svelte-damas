<script lang="ts">
  import _ from 'lodash'

  const game: number[] = _.fill(Array(64), 0)
</script>

<style>
  .table {
    width: 650px;
    height: 640px;
    display: grid;
    grid-template-rows: repeat(8, 1fr);
    grid-template-columns: repeat(8, 1fr);
  }

  .cell:nth-child(2n + 1) {
    background-color: #AF8460;
  }

  .cell:nth-child(2n) {
    background-color: #E8D2AF;
  }
</style>

<div class="table">
  {#each game as cell, i}
    <div class={`cell row-${Math.floor(i / 8) + 1}`}>
      {cell}
    </div>
  {/each}
</div>
