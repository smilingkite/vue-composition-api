<template>
	<!--	met dank aan: https://github.com/Ijee/Game-of-Life-Vue2/ -->
	<div>
		<h1>game of life</h1>
		<div
			class="game-grid columns"
			@mousedown="isMouseDown = true"
			@mouseup="isMouseDown = false"
			@mouseleave="isMouseDown = false">
			<div
				v-for="(col, indexX) in gridList"
				:key="indexX"
				class="game-column">
				<GOLcell
					v-for="(isAlive, indexY) in col"
					:key="indexY"
					:status-obj="isAlive"
					:x-pos="indexX"
					:y-pos="indexY"
					:is-mouse-down="isMouseDown"
					@wasUpdated="updateCellCount"
				/>
			</div>
		</div>
<!--			:class="isRunning ? 'green' : 'red'"-->
		<button
			class="button"
			title="play"
			@click="play" >play</button>
	</div>
</template>

<script>
	import {createComponent, computed, ref, reactive} from '@vue/composition-api';
	import GOLcell from '../components/GOLcell'; // ? waarom werkt @/components syntax niet?
	import DemoImage from '@/components/DemoImage'; // waarom werkt @/components syntax hier?
	import {setInterval, clearInterval} from 'timers';
	// import useFileDnD from '../composables/use-file-dnd';
	// import useGlobalDnD from '../composables/use-global-dnd';

	export default createComponent({
		name: 'Game',
		components: {
			GOLcell,
			DemoImage
		},
		setup() {

			let width = 46;
			let height = 20;
			let isMouseDown = reactive(false);
			let isRunning = reactive(false);
			const cellCalc = () => {
				let gridList = [];
				for (let i = 0; i < width; i++) {
					gridList[i] = [];
					for (let j = 0; j < height; j++) {
						gridList[i][j] = {isAlive: false};
					}
				}
				return reactive(gridList);
			};
			let gridList = ref(cellCalc());

			/**
			 * Changes the 'isAlive' object property
			 * of a specific cell to the one requested
			 * in the param.
			 *
			 * @param {number} x - the x position
			 * @param {number} y - the y position
			 * @param {boolean} bool - the new boolean
			 */
			const setCell = (x, y, bool) => {
				if (gridList[x][y].isAlive != bool) {
					gridList[x][y].isAlive = bool;
				}
				// let row = this.gridList[x];
				// row.splice(y, 1, {isAlive: true});
				// this.gridList.splice(x, 1, row);
			};

			const update = () => {
				let tempArr = [];
				for (let i = 0; i < width; i++) {
					tempArr[i] = [];
					for (let j = 0; j < height; j++) {
						let status = gridList[i][j].isAlive;
						let neighbours = getNeighbours(i, j);
						let result = false;
						// Rule 1:
						// Any live cell with fewer than two live neighbours dies,
						// as if by under population
						if (status && neighbours < 2) {
							result = false;
						}
						// Rule 2:
						// Any live cell with two or three neighbours lives on
						// to the next generation
						if ((status && neighbours == 2) || neighbours == 3) {
							result = true;
						}
						// Rule 3:
						// Any live cell with more than three live neighbours dies,
						// as if by overpopulation
						if (status && neighbours > 3) {
							result = false;
						}
						// Rule 4:
						// Any dead cell with exactly three live neighbours becomes
						// a live cell, as if by reproduction
						if (!status && neighbours == 3) {
							result = true;
						}
						tempArr[i][j] = result;
					}
				}
				// set new gridList content
				for (let i = 0; i < width; i++) {
					for (let j = 0; j < height; j++) {
						setCell(i, j, tempArr[i][j]);
					}
				}
			};

			/**
			 * Returns the amount of neighbours for
			 * a specific cell on the grid.
			 *
			 * @param {number} posX - the x position
			 * @param {number} posY - the Y position
			 * @return {number} neighbours - amount of neighbours
			 */
			const getNeighbours = (posX, posY) => {
				let neighbours = 0;
				if (posX <= width && posY <= height) {
					for (let offsetX = -1; offsetX < 2; offsetX++) {
						for (let offsetY = -1; offsetY < 2; offsetY++) {
							let newX = posX + offsetX;
							let newY = posY + offsetY;
							// check if offset is:
							// on current cell, out of bounds and if isAlive
							// for cell true
							if (
								(offsetX != 0 || offsetY != 0) &&
								newX >= 0 &&
								newX < this.width &&
								newY >= 0 &&
								newY < this.height &&
								gridList[posX + offsetX][posY + offsetY].isAlive == true
							) {
								neighbours++;
							}
						}
					}
				}
				return neighbours;
			};
			const restartInterval = () => {
				clearInterval(this.intervalID);
				if (isRunning) {
					this.intervalID = setInterval(
						update(),
						50000 / 20,
						'nextStep'
					);
				}
			}
			const play = () => {
				isRunning = !isRunning;
				restartInterval();
				console.log('play...')
			}
			return {
				width,
				height,
				gridList,
				//
				// // A prop that gets used by the app-cell component (drag)
				isMouseDown,
				//
				// // Methods exported
				setCell,
				update,
				restartInterval,
				clearInterval,
				play

			};
		}
	});
</script>

<style scoped>

	h1 {
		padding: 15px;
		font-size: 2em;
	}

	.game-grid {
		border-top: 1px solid #1a0707;
		border-left: 1px solid #1a0707;
		display: flex;
		flex: 1;
		justify-content: center;
	}
	.game-column {
		flex: 1;
		display: flex;
		justify-content: center;
		padding: 0;
		margin: 0 auto;
		flex-direction: column;
	}
</style>
