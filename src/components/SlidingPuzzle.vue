<template>
    <div>
        <div><h1>Sliding Puzzle</h1></div>
        
        <div v-if="resolved">
            <div v-if="counter==0">Press shuffle to start</div>
            <div v-else>Resolved in {{ counter }} moves</div>
        </div>
        <div v-else>Moves {{ counter }}</div>

        <div
            class="container"
            :key="changed"
            :style="{
                width: (cols*100) + 'px',
                height: (rows*100) + 'px'
            }"

        >
            <div class="row" v-for="(nx,idx) in rows" :key=" idx+10000 ">
                <div
                    v-for="(ny,idy) in cols" :key=" (idx*cols)+idy+1 "
                    :style="{
                        top: (idx*100) + 'px',
                        left: (idy*100) + 'px'
                    }"
                    :class="{
                        'col': true,
                        'goright': (puzzle[idx][idy].dir == 4),
                        'goleft': (puzzle[idx][idy].dir == 3),
                        'godown': (puzzle[idx][idy].dir == 2),
                        'goup': (puzzle[idx][idy].dir == 1),
                    }"
                    @click="clicked(idx, idy)"
                    @animationend="adone(idx, idy)"
                >
                    <div v-if="isNotEmpty(idx, idy)" class="box">
                        {{ puzzle[idx][idy].text }}
                    </div>
                </div>
            </div>
        </div>
        
        <div>
            <button class="button" :disabled="!resolved" @click="shuffle">Shuffle</button>
        </div>
    </div>
</template>

<script>
export default {
    name: 'SlidingPuzzle',
    data () {
        return {
            changed: 1,
            rows: 3,
            cols: 3,
            puzzle: [],
            shuffles: 100,
            counter: 0,
            resolved: true,
            empty: {
                x: 2,
                y: 2
            },
            directions: [
                { ax: -1, ay: 0,  dir: 1 },     // Up
                { ax: 1,  ay: 0,  dir: 2 },     // Down
                { ax: 0,  ay: -1, dir: 3 },     // Left
                { ax: 0,  ay: 1,  dir: 4 },     // Right
            ]

        }
    },

    /**
     * Created
     */
    created() {
        // Initialize puzzle
        this.init()
    },

    /**
     * Methods
     */
    methods: {
        /**
         * Initialize puzzle
         */
        init() {
            // Create board as 2D -array
            for (let x=0; x < this.rows; x++) {
                this.puzzle[x] = [];
                for (let y=0; y < this.cols; y++) {
                    let c = (x * this.cols) + y + 1
                    this.puzzle[x][y] = {
                        text: (this.cols * this.rows !== c ? c : null),
                        dir: 0
                    }
                }
            }
            // Last cell is empty
            this.empty = {
                x: this.rows-1,
                y: this.cols-1
            }
        },

        /**
         * Shuffle puzzle
         */
        shuffle() {
            this.init()

            this.resolved = false
            this.counter = 0

            let r = 0
            let b = null

            // Make random moves "this.shuffles" times
            for (let z=0; z < this.shuffles; z++) {
                do {
                    // Generate a random direction 1 .. 4
                    r = Math.floor(Math.random() * 4) + 1
                    // Can we move to the direction?
                    b = this.checkIfMoveable(r, this.empty.x, this.empty.y)
                } while (b === null)

                // Swap tiles
                this.swapTiles(b.x, b.y)
            }
            this.changed *= -1
        },

        /**
         * Swap tile with empty tile
         * [x][y] is the tile
         */
        swapTiles(x, y) {
            this.puzzle[this.empty.x][this.empty.y] = {
                text: this.puzzle[x][y].text,
                dir: 0
            }
            this.puzzle[x][y] = { text: null, dir: 0 }
            this.empty.x = x
            this.empty.y = y
        },

        /**
         * Check if puzzle numbers are in right order
         */
        checkPuzzle() {
            for (let x=0; x < this.rows; x++) {
                for (let y=0; y < this.cols; y++) {
                    if (this.isNotEmpty(x, y)) {
                        let c = (x * this.cols) + y + 1
                        if (this.puzzle[x][y].text !== c) {
                            // Found number in wrong place, puzzle is not done
                            return false
                        }
                    }
                }
            }
            return true
        },

        /**
         * Is tile non empty ?
         * Return true if non empty, false otherwise
         */
        isNotEmpty(x, y) {
            if ((x >= 0) && (y >= 0) && (x < this.rows) && (y < this.cols)) {
                // Insize puzzle. Is it non-empty?
                return !this.isEmpty(x, y)
            }
            return false
        },

        /**
         * Check if the tile in given direction has a number
         */
        checkIfMoveable(dir, x, y) {
            for (let a of this.directions) {
                if ((dir === a.dir) && (this.isNotEmpty(x + a.ax, y + a.ay))) {
                    return {
                        x: x + a.ax,
                        y: y + a.ay,
                        dir: a.dir 
                    }
                }
            }
            return null
        },

        /**
         * Check if tile is empty
         */
        isEmpty(x, y) {
            return ((this.empty.x === x) && (this.empty.y === y))
        },

        /**
         * Check where the empty tile is
         */
        getDirToEmpty(x, y) {
            for (let a of this.directions) {
                if (this.isEmpty(x + a.ax, y + a.ay)) {
                    return a.dir
                }
            }
            return null
        },

        /**
         * User clicked on tile
         */
        clicked(idx, idy) {
            if (this.resolved) {
                // Resolved
                return
            }
            if (this.puzzle[idx][idy].dir > 0) {
                // Still animating ..
                return
            }
            if (this.isEmpty(idx, idy)) {
                // Empty tile ..
                return
            }

            let d = this.getDirToEmpty(idx, idy)
            if (d) {
                // Found empty tile at direction d
                // This will trigger to add an animation class to tile.
                // After the animation stops it will trigger an "adone" event.
                this.puzzle[idx][idy].dir = d

                this.counter += 1
                this.changed *= -1
            }
        },

        /**
         * Animation of tile movement done. Now swap the tiles
         */
        adone(idx, idy) {
            this.puzzle[idx][idy].dir = 0
            this.swapTiles(idx, idy)

            this.changed *= -1

            if (this.checkPuzzle()) {
                this.resolved = true
            }
        }
    }

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.button {
    width: 100px;
    height: 30px;
    font-size: 20px;
    margin-top: 25px;
}
.container {
    display: table;
    width: auto;
    background-color: #eee;
    border: 0;
    border-spacing: 2px;
    margin: auto auto;
    position: relative;
}
.row {
    display: table-row;
    width: auto;
    clear: both;
}
.col {
    float: left;
    display: table-column;
    background-color: #eee;
    width: 100px;
    height: 100px;
    position: absolute;
}
.box {
    border: 2px solid #eee;
    width: 96px;
    height: 96px;
    background-color: #ccc;
    font-size: 82px;
    color: green;
    cursor: pointer;
}
.goright, .goleft, .godown, .goup {
    z-index: 100;
    animation-duration: 0.1s;
    animation-fill-mode: forwards;
}
.goright {
    animation-name: kf-goright;
}
@keyframes kf-goright {
    from {
        transform: translateX(0%)
    }
    to {
        transform: translateX(100%)
    }
}
.goleft {
    animation-name: kf-goleft;
}
@keyframes kf-goleft {
    from {
        transform: translateX(0%)
    }
    to {
        transform: translateX(-100%)
    }
}
.godown {
    animation-name: kf-godown;
}
@keyframes kf-godown {
    from {
        transform: translateY(0%)
    }
    to {
        transform: translateY(100%)
    }
}
.goup {
    animation-name: kf-goup;
}
@keyframes kf-goup {
    from {
        transform: translateY(0%)
    }
    to {
        transform: translateY(-100%)
    }
}
</style>
