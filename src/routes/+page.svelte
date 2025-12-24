<script lang="ts">
	import { type Board, dimension, newBoard, pieceImgs, type Piece, type Colour } from '$lib/pieces';
	import {byEco, type Opening, ecoCodes} from "$lib/openings"
	let board = $state<Board>(newBoard());

    let turn = $state<Colour>("white")
	let selectedPiece = $state<Piece>();
	let possibleMoves = $derived(selectedPiece?.possibleMoves(board));
	// TODO: make move then show possible opening continuations/name
	let selectedOpening = $state<Opening | undefined>(byEco.get(ecoCodes[0]))
	let moveIdx = $state(-1)
</script>

<div>
	<select onchange={({currentTarget}) => {
		const opening = byEco.get(currentTarget.value)
		if (opening) {
			selectedOpening = opening
			board = newBoard()
			moveIdx = -1
		}
	}}>
		{#each byEco as [eco, {name}]}
			<option value={eco}>{eco} {name}</option>
		{/each}
	</select>
</div>
<div>
	<button disabled={moveIdx < 0} onclick={() => moveIdx--}>Prev</button>
	<button disabled={!selectedOpening || moveIdx >= selectedOpening.uci.length-1 } onclick={() => {
		moveIdx++
		const move = selectedOpening?.uci.at(moveIdx)
		if (!move) return;
		const {start, end} = move
		board[end.row][end.col] = board[start.row][start.col]
		board[start.row][start.col] = null
		}}>Next</button>
</div>
<div>{moveIdx} {selectedOpening?.uci}</div>
<div id="board-container">
<div id="board">
	<div></div>
	{#each { length: dimension } as _, i}
	<div class="board-label">{String.fromCharCode(65+i)}</div>
	{/each}
	
	{#each { length: dimension } as _, row}
		<div class="board-label">{dimension - row}</div>
		{#each { length: dimension } as _, col}
			
			{@const piece = board.at(row)?.at(col)}
			<div class="square">
				<button
					
					style="height:100%;width:100%"
					style:background={possibleMoves?.find((p) => p.col===col && p.row === row) ? "lime" : (row + col) % 2 === 0 ? 'white' : 'rgba(0,0,0,0.5)'}
					onclick={() => {
                        // if (piece?.colour === turn) {
                        //     selectedPiece = piece
                        //     return
                        // } 
                        // if (selectedPiece && possibleMoves?.find((p) => p.col===col && p.row === row)) {
                        //     board[selectedPiece.position.row][selectedPiece.position.col] = null
                        //     selectedPiece.position = {row, col}
                        //     board[row][col] = selectedPiece
                        //     selectedPiece = undefined
                        //     turn = turn === "black" ? 'white': "black"
                        // }
                    }}
				>
					{#if piece}
						<img height="100%" width="100%" src={pieceImgs[piece.colour][piece.tag]} >
					{/if}
				</button>
			</div>
		{/each}
	{/each}
</div>
</div>

<style>
	#board {
		display: grid;
		grid-template-columns: repeat(9, 10vmin);
		grid-template-rows: repeat(9, 10vmin);
	}
	.square {
		border: 1px solid black;
	}
	.board-label {
		text-align: center;
		align-content: center;
	}
	#board-container {
		display: flex;
		justify-content: center;
	}
</style>
