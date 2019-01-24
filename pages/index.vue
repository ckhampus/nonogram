<template>
  <div class="flex w-screen h-screen flex-col items-center p-10">
    <div class="area flex flex-col bg-grey-lighter border border-black">
      <div class="flex justify-end">
        <div class="flex flex-row items-end bg-grey-light">
          <div
            class="column"
            :class="{
              'opacity-25': isColumnClueComplete(index),
              'text-red': isInvalidColumn(index)
              }"
            v-for="(clues, index) in columns"
            :key="index"
          >
            <div class="cell" v-for="(clue, index) in clues" :key="index">{{clue}}</div>
          </div>
        </div>
      </div>
      <div class="flex">
        <div class="flex flex-col items-end bg-grey-light">
          <div
            class="row"
            :class="{
              'opacity-25': isRowClueComplete(index),
              'text-red': isInvalidRow(index)
              }"
            v-for="(clues, index) in rows"
            :key="index"
          >
            <div class="cell" v-for="(clue, index) in clues" :key="index">{{clue}}</div>
          </div>
        </div>
        <!-- GRID -->
        <div class="flex flex-col" ref="grid" @touchstart="onMouseDown">
          <div class="row" v-for="i in height" :key="i">
            <input
              class="cell grid-cell border-t border-l border-grey-darkest"
              :class="{
                'bg-grey-light': isColumnClueComplete(j-1) && isRowClueComplete(i-1),
              }"
              v-for="j in width"
              :key="i+j"
              type="checkbox"
              v-model="selected[i-1][j-1]"
            >
          </div>
        </div>
      </div>
    </div>
    <p class="text-2xl p-10" v-if="goal === marked">Yay!</p>
  </div>
</template>

<script>
export default {
  data() {
    const width = 7;
    const height = 8;

    return {
      width,
      height,
      columns: [[3, 1], [4, 1], [8], [3, 1], [5], [4], [4]],
      rows: [[1, 1], [3], [3], [4], [6], [5], [1, 1, 3], [6]],
      goal: "10100001110000111000001111000111111001111110101110111111",
      selected: new Array(height)
        .fill(false)
        .map(() => new Array(width).fill(false)),
      isDragging: false,
      setCellTo: false
    };
  },
  mounted() {
    window.addEventListener('touchmove', this.onMouseMove);
    window.addEventListener('touchend', this.onMouseUp);
  },
  computed: {
    clueRowLength() {
      return this.rows.reduce((prev, clue) => Math.max(clue.length, prev), 0);
    },
    clueColumnLength() {
      return this.rows.reduce((prev, clue) => Math.max(clue.length, prev), 0);
    },
    marked() {
      return this.selected
        .map(row => row.map(v => (v ? "1" : "0")).join(""))
        .join("");
    }
  },
  methods: {
    onMouseDown(event) {
      this.isDragging = true;

      if (event.target instanceof HTMLInputElement) {
        this.setCellTo = !event.target.checked;
      }
    },
    onMouseMove(event) {
      if (this.isDragging) {
        const {clientX, clientY} = event.touches[0];
        const {top, left} = this.$refs.grid.getBoundingClientRect();
        const gridX = clamp(Math.floor((clientX - left) / 30), 0, this.width - 1)
        const gridY = clamp(Math.floor((clientY - top) / 30), 0, this.height -1)
        this.setSelection(gridX, gridY);
      }
    },
    onMouseUp(event) {
      console.log('onMouseUp')
      this.isDragging = false;
    },
    setSelection(x, y) {
      this.selected[y][x] = this.setCellTo;
      this.$forceUpdate();
    },
    isColumnClueComplete(column) {
      return isClueComplete(this.columns[column], this.getGridColumn(column));
    },
    isRowClueComplete(row) {
      return isClueComplete(this.rows[row], this.getGridRow(row));
    },
    isInvalidColumn(column) {
      const clues = this.columns[column];
      const selection = countArray(this.getGridColumn(column));
      return sumArray(clues) < sumArray(selection) || selectionToLarge(clues, selection);
    },
    isInvalidRow(row) {
      const clues = this.rows[row];
      const selection = countArray(this.getGridRow(row));
      return sumArray(clues) < sumArray(selection) || selectionToLarge(clues, selection);
    },
    getGridColumn(column) {
      return this.selected.map(row => row[column]);
    },
    getGridRow(row) {
      return this.selected[row];
    }
  }
};

function clamp(value, min, max) {
  return Math.max(min, Math.min(value, max));
}

function isClueComplete(clue, selected) {
  return compareArray(clue, countArray(selected));
}

function sumArray(array) {
  return array.reduce((acc, curr) => acc + curr, 0)
}

function selectionToLarge(clues, selection) {
  const length = Math.min(clues.length, selection.length);

  for (let i = 0; i < length; i++) {
    if (selection[i] > clues[i]) return true;
  }

  return false;
}

function countArray(array) {
  const result = [];
  let count = 0;
  for (let i = 0; i < array.length; i++) {
    const value = array[i];

    if (value) {
      count++;
    } else {
      result.push(count);
      count = 0;
    }
  }

  result.push(count);

  return result.filter(v => !!v);
}

function compareArray(array1, array2) {
  return (
    array1.length === array2.length &&
    array1.every((value, index) => value === array2[index])
  );
}
</script>

<style>
.row {
  display: flex;
}

.grid-row {
  display: flex;
}

.cell {
  display: flex;
  width: 30px;
  height: 30px;
  justify-content: center;
  align-items: center;
}

.grid-cell {
  appearance: none;
}

.grid-cell:checked {
  background-color: #22292F;
}

.grid-cell:focus {
  outline: none;
}

.grid-clue {
  background: #ccc;
}
</style>
