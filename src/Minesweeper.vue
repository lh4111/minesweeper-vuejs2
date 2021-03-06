<template>
<div class="minesweeper">
<h1>
    {{game.state}}
    <button v-if="game.state != 'new'"
        v-on:click="restart">

        Restart
    </button>
</h1>
<h3>{{getTime(game.timer.ms)}}</h3>
<ol id="board" class="unselectable">
    <li class="row" v-bind:width="rowWidth(game.board.cells.length)" v-for="(row, rindex) in game.board.cells">
        <div v-for="(cell, cindex) in row"
            v-bind:class="classes(cell)"
            v-bind:data-row="rindex"
            v-bind:data-col="cindex"
            v-on:click="clickCell"
            v-on:contextmenu="flag">

            <span v-if="cell.visible" class="cell-outer unselectable">
                <span v-if="isEmptyOrMine(cell)" class="unselectable">&nbsp;</span>
                <span v-if="showCell(cell)" class="unselectable">{{cell.nearCount}}</span>
            </span>
            <span v-else class="cell-outer unselectable">
                <span>&nbsp;</span>
            </span>
        </div>
    </li>
</ol>
</div>

</template>

<script>
const Game = require('./Game.js');
const game = new Game({skill: 'intermediate'});
const getRowCol = (target) => {
    let data = target.dataset;

    return {
        row: parseInt(data.row),
        col: parseInt(data.col),
    };
};

export default {
    data: {
        game: game
    },

    methods: {
        classes: (cell) => {
            let classes = 'cell unselectable';

            if (cell.flagged) {
                classes += ' flagged';
            }
            if (!cell.visible) {
                classes += ' not-visible';
            } else {
                if (cell.mine) {
                    classes += ' mine';
                }
                classes += ' visible';
            }

            return classes;
        },
        rowWidth: (boardWidth) => {
            return boardWidth * 27;
        },
        getTime: (ms) => {
            let time = game.timer.getTime();
            return `${time.hours}:${time.minutes}:${time.seconds}:${time.ms}`;
        },
        isEmptyOrMine: (cell) => {
            return cell.nearCount === 0 || cell.mine;
        },
        showCell: (cell) => {
            return cell.nearCount !== 0 && !cell.mine;
        },
        clickCell: function (e) {
            e.preventDefault();

            game.onCellSelect(getRowCol(e.target));
            this.game = game;
        },
        flag: function (e) {
            e.preventDefault();

            game.onCellFlag(getRowCol(e.target));
            this.game = game;
        },
        restart: function (e) {
            e.preventDefault();

            game.onRestart();
            this.game = game;
        }
    }
};

</script>

<style lang="sass">
.minesweeper {
    font-family: 'Dosis', sans-serif;
}
.row {
    height: 27px;
}
.cell-outer {
    height: 25px;
    width: 25px;
    display: inline-block;
}
.cell {
    height: 25px;
    width: 25px;
    border: 1px solid black;
    display: inline-block;
    text-align: center;
    z-index: 900;
    border-radius: 2px;

    -webkit-transform: rotate3d(45,45,0,0deg);
    -webkit-transform-style: preserve-3d;
    -webkit-backface-visibility: hidden;
    -webkit-transition: all .3s ease-in-out;

    transform: rotate3d(45,45,0,0deg);
    transform-style: preserve-3d;
    backface-visibility: hidden;
    transition: all .3s ease-in-out;

    &.visible {
        background: rgb(255,255,255);
        background: -moz-linear-gradient(45deg, rgba(255,255,255,1) 0%, rgba(243,243,243,1) 50%, rgba(237,237,237,1) 51%, rgba(255,255,255,1) 100%);
        background: -webkit-linear-gradient(45deg, rgba(255,255,255,1) 0%,rgba(243,243,243,1) 50%,rgba(237,237,237,1) 51%,rgba(255,255,255,1) 100%);
        background: linear-gradient(45deg, rgba(255,255,255,1) 0%,rgba(243,243,243,1) 50%,rgba(237,237,237,1) 51%,rgba(255,255,255,1) 100%);
        filter: progid:DXImageTransform.Microsoft.gradient( startColorstr='#ffffff', endColorstr='#ffffff',GradientType=1 );
        &.mine {
            background: url('mine.svg');
        }
    }
    &.not-visible {
        background: #fceabb;
        background: -moz-linear-gradient(-45deg, #fceabb 0%, #fccd4d 50%, #f8b500 51%, #fbdf93 100%);
        background: -webkit-linear-gradient(-45deg, #fceabb 0%,#fccd4d 50%,#f8b500 51%,#fbdf93 100%);
        background: linear-gradient(135deg, #fceabb 0%,#fccd4d 50%,#f8b500 51%,#fbdf93 100%);
        filter: progid:DXImageTransform.Microsoft.gradient( startColorstr='#fceabb', endColorstr='#fbdf93',GradientType=1 );
    }
    &.flagged {
        background: #f3c5bd;
        background: -moz-linear-gradient(-45deg, #f3c5bd 0%, #e86c57 50%, #ea2803 51%, #ff6600 75%, #c72200 100%);
        background: -webkit-linear-gradient(-45deg, #f3c5bd 0%,#e86c57 50%,#ea2803 51%,#ff6600 75%,#c72200 100%);
        background: linear-gradient(135deg, #f3c5bd 0%,#e86c57 50%,#ea2803 51%,#ff6600 75%,#c72200 100%);
        filter: progid:DXImageTransform.Microsoft.gradient( startColorstr='#f3c5bd', endColorstr='#c72200',GradientType=1 );
    }
    &.not-visible,
    &.flagged {
        z-index: 800;
        border-radius: 2px;

        -webkit-transform: rotate3d(45,45,0,-360deg);
        -webkit-transform-style: preserve-3d;
        -webkit-backface-visibility: hidden;
        -webkit-transition: all .3s ease-in-out;

        transform-style: preserve-3d;
        backface-visibility: hidden;
        transition: all .3s ease-in-out;
        transform: rotate3d(45,45,0,-360deg);
    }
}

.unselectable {
  -webkit-user-select: none;  /* Chrome all / Safari all */
  -moz-user-select: none;     /* Firefox all */
  -ms-user-select: none;      /* IE 10+ */
  user-select: none;          /* Likely future */      
}

span {
    pointer-events: none;
}

li {
    list-style-type: none;
}

</style>
