<script>
import { writable } from "svelte/store";

const store = writable(localStorage.getItem("store") || "");

store.subscribe(val => localStorage.setItem("store", val))

export let teams = [];
export let league_size = 2;

function setLeagueSize() {
	teams = [];
	for( let i = 0; i < league_size; i++ ) {
		teams.push( {name: 'ABC', anime: []} );
	}
	saveChanges();
}

function saveChanges() {
	$store = JSON.stringify( teams );
	console.log( $store );
}
function loadChanges() {
	teams = JSON.parse( $store );
	console.log( $store );
}
/* ----- END SCRIPT ----- */
</script>

<nav>
	<h1>Anime Fantasy League</h1>
	<button on:click={saveChanges}>Save</button>
	<button on:click={loadChanges}>Load</button>
</nav>

<main>
	<p>Number of people in your league:</p>
	<input name="league_size" bind:value={league_size} />
	<button on:click={setLeagueSize}>Set</button>
	<div class="teams">
		{#each teams as team}
		<div class="team">
			<h2><input bind:value={team.name} /></h2>
		</div>
		{/each}
	</div>
</main>

<style>
/* ----- END HTML ----- */

/* Palette: https://www.colourlovers.com/palette/4780148/Palette_for_2021 */
	
	main {
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		text-align: center;
		color: #607AA8;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	.teams {
		display: flex;
		flex-direction: row;
	}

	.team {
		flex-grow: 1;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>