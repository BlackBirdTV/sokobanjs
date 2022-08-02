<script setup lang="ts">

import { ref } from 'vue';

class Block {
    x: number
    y: number
    symbol: string
    targetx: number
    targety: number
    targetsymbol: string
    constructor (x: number, y: number, symbol: string, targetx: number, targety: number, targetsymbol: string) {
        this.x = x;
        this.y = y;
        this.symbol = symbol;
        this.targetx = targetx;
        this.targety = targety;
        this.targetsymbol = targetsymbol;
    }
}

const PLAYER = '😁';
const EMPTY = '⬛';

class Level {
    playerPos: number[]
    blocks: Block[]
    constructor(playerPos: number[], blocks: Block[]) {
        this.playerPos = playerPos
        this.blocks = blocks;
    }
}

let levels = [
    new Level([0, 0], [
        new Block(5, 5, '🟪', 6, 6, '🟣'),
        new Block(5, 4, '🟩', 6, 5, '🟢')
    ]),
    new Level([5, 5], [
        new Block(5, 6, '🟪', 5, 7, '🟣'),
        new Block(5, 4, '🟩', 5, 3, '🟢'),
        new Block(4, 5, '🟥', 3, 5, '🔴'),
        new Block(6, 5, '🟨', 7, 5, '🟡'),
    ])
];

let currentLevel = new Level([0, 0], []);
let blocksLeft = 0;
let levelIdx = 0;

let gameField = ref([['']]);

function redraw() {
    gameField.value = [
    [EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY],
    [EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY],
    [EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY],
    [EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY],
    [EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY],
    [EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY],
    [EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY],
    [EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY],
    [EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY],
    [EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY, EMPTY],
];
    for (let i = 0; i < currentLevel.blocks.length; i++) {
        const block = currentLevel.blocks[i];
        gameField.value[block.targety][block.targetx] = block.targetsymbol
        gameField.value[block.y][block.x] = block.symbol
    }
    gameField.value[currentLevel.playerPos[1]][currentLevel.playerPos[0]] = PLAYER;
}

function update(event: KeyboardEvent) {
    const key = event.key;
    if ((key == 'ArrowUp' || key == 'w') && currentLevel.playerPos[1] > 0) {
        currentLevel.playerPos[1] -= 1
        for (let i = 0; i < currentLevel.blocks.length; i++) {
            const block = currentLevel.blocks[i];
            if (block.x == currentLevel.playerPos[0] && block.y == currentLevel.playerPos[1]) {
                const symbol = gameField.value[block.y - 1][block.x ]
                if (symbol != EMPTY && symbol != block.targetsymbol) {
                    currentLevel.playerPos[1] += 1
                }
                else {
                    block.y -= 1;
                }
            }
        }
    }
    else if ((key == 'ArrowDown' || key == 's') && currentLevel.playerPos[1] < 9) {
        currentLevel.playerPos[1] += 1
        for (let i = 0; i < currentLevel.blocks.length; i++) {
            const block = currentLevel.blocks[i];
            if (block.x == currentLevel.playerPos[0] && block.y == currentLevel.playerPos[1]) {
                const symbol = gameField.value[block.y + 1][block.x]
                if (symbol != EMPTY && symbol != block.targetsymbol) {
                    currentLevel.playerPos[1] -= 1
                }
                else {
                    block.y += 1;
                }
            }
        }
    }
    else if ((key == 'ArrowLeft' || key == 'a') && currentLevel.playerPos[0] > 0) {
        currentLevel.playerPos[0] -= 1
        for (let i = 0; i < currentLevel.blocks.length; i++) {
            const block = currentLevel.blocks[i];
            if (block.x == currentLevel.playerPos[0] && block.y == currentLevel.playerPos[1]) {
                const symbol = gameField.value[block.y][block.x - 1]
                if (symbol != EMPTY && symbol != block.targetsymbol) {
                    currentLevel.playerPos[0] += 1
                }
                else {
                    block.x -= 1;
                }
            }
        }
    }
    else if ((key == 'ArrowRight' || key == 'd') && currentLevel.playerPos[0] < 9) {
        currentLevel.playerPos[0] += 1
        for (let i = 0; i < currentLevel.blocks.length; i++) {
            const block = currentLevel.blocks[i];
            if (block.x == currentLevel.playerPos[0] && block.y == currentLevel.playerPos[1]) {
                const symbol = gameField.value[block.y][block.x + 1]
                if (symbol != EMPTY && symbol != block.targetsymbol) {
                    currentLevel.playerPos[0] -= 1
                }
                else {
                    block.x += 1;
                }
            }
        }
    }
    for (let i = 0; i < currentLevel.blocks.length; i++) {
        const block = currentLevel.blocks[i];
        if (block.x == block.targetx && block.y == block.targety) {
            currentLevel.blocks.splice(i, 1);
            blocksLeft -= 1
        }
    }
    if (blocksLeft == 0) {
        showWinningScreen.value = true;        
    }
    redraw()
}

const restart = () => loadLevel(levelIdx)

function loadLevel(i: number) {
    levelIdx = i;
    currentLevel = JSON.parse(JSON.stringify(levels[i]));
    blocksLeft = currentLevel.blocks.length
    redraw()
}

function nextLevel() {
    showWinningScreen.value = false;
    loadLevel(levelIdx + 1 > levels.length - 1 ? 0 : levelIdx + 1)
}

let showWinningScreen = ref(false);

document.addEventListener('keyup', update);

loadLevel(0)
</script>

<template>
<a @click="restart()">Restart</a>
    <div>🟦🟦🟦🟦🟦🟦🟦🟦🟦🟦🟦🟦<br>
🟦{{ gameField[0][0] }}{{ gameField[0][1] }}{{ gameField[0][2] }}{{ gameField[0][3] }}{{ gameField[0][4] }}{{ gameField[0][5] }}{{ gameField[0][6] }}{{ gameField[0][7] }}{{ gameField[0][8] }}{{ gameField[0][9] }}🟦<br>
🟦{{ gameField[1][0] }}{{ gameField[1][1] }}{{ gameField[1][2] }}{{ gameField[1][3] }}{{ gameField[1][4] }}{{ gameField[1][5] }}{{ gameField[1][6] }}{{ gameField[1][7] }}{{ gameField[1][8] }}{{ gameField[1][9] }}🟦<br>
🟦{{ gameField[2][0] }}{{ gameField[2][1] }}{{ gameField[2][2] }}{{ gameField[2][3] }}{{ gameField[2][4] }}{{ gameField[2][5] }}{{ gameField[2][6] }}{{ gameField[2][7] }}{{ gameField[2][8] }}{{ gameField[2][9] }}🟦<br>
🟦{{ gameField[3][0] }}{{ gameField[3][1] }}{{ gameField[3][2] }}{{ gameField[3][3] }}{{ gameField[3][4] }}{{ gameField[3][5] }}{{ gameField[3][6] }}{{ gameField[3][7] }}{{ gameField[3][8] }}{{ gameField[3][9] }}🟦<br>
🟦{{ gameField[4][0] }}{{ gameField[4][1] }}{{ gameField[4][2] }}{{ gameField[4][3] }}{{ gameField[4][4] }}{{ gameField[4][5] }}{{ gameField[4][6] }}{{ gameField[4][7] }}{{ gameField[4][8] }}{{ gameField[4][9] }}🟦<br>
🟦{{ gameField[5][0] }}{{ gameField[5][1] }}{{ gameField[5][2] }}{{ gameField[5][3] }}{{ gameField[5][4] }}{{ gameField[5][5] }}{{ gameField[5][6] }}{{ gameField[5][7] }}{{ gameField[5][8] }}{{ gameField[5][9] }}🟦<br>
🟦{{ gameField[6][0] }}{{ gameField[6][1] }}{{ gameField[6][2] }}{{ gameField[6][3] }}{{ gameField[6][4] }}{{ gameField[6][5] }}{{ gameField[6][6] }}{{ gameField[6][7] }}{{ gameField[6][8] }}{{ gameField[6][9] }}🟦<br>
🟦{{ gameField[7][0] }}{{ gameField[7][1] }}{{ gameField[7][2] }}{{ gameField[7][3] }}{{ gameField[7][4] }}{{ gameField[7][5] }}{{ gameField[7][6] }}{{ gameField[7][7] }}{{ gameField[7][8] }}{{ gameField[7][9] }}🟦<br>
🟦{{ gameField[8][0] }}{{ gameField[8][1] }}{{ gameField[8][2] }}{{ gameField[8][3] }}{{ gameField[8][4] }}{{ gameField[8][5] }}{{ gameField[8][6] }}{{ gameField[8][7] }}{{ gameField[8][8] }}{{ gameField[8][9] }}🟦<br>
🟦{{ gameField[9][0] }}{{ gameField[9][1] }}{{ gameField[9][2] }}{{ gameField[9][3] }}{{ gameField[9][4] }}{{ gameField[9][5] }}{{ gameField[9][6] }}{{ gameField[9][7] }}{{ gameField[9][8] }}{{ gameField[9][9] }}🟦<br>
🟦🟦🟦🟦🟦🟦🟦🟦🟦🟦🟦🟦</div>
<div :style="{bottom: showWinningScreen ? '10%' : '-80%'}" class="winning-screen">
<h1>Level Cleared</h1>
<button @click="nextLevel()">Continue</button>
</div>
</template>

<style>
body, html {
    margin: 0;
    width: 100vw;
    height: 100vh;
    display: grid;
    place-items: center;
    background: #31373D;
}

div:not(.winning-screen) {
    font-size: 2rem;
    line-height: 2rem;
}

a {
    color: #509CD7;
    text-decoration: none;
    cursor: pointer;
}

a:hover {
    color: #5489b1;
}

.winning-screen {
    position: fixed;
    width: 80%;
    height: 80%;
    left: 10%;
    background-color: #435363;
    border-radius: 5rem;
    transition: bottom 500ms ease-in-out;
    
    display: flex;
    flex-direction: column;
    align-items: center;
}

button {
    width: 20rem;
    height: 5rem;
    background-color: #509CD7;
    border-radius: 9999px;
}
</style>
