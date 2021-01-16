<script>
import { writable } from "svelte/store";

const store = writable(localStorage.getItem("store") || "");

store.subscribe(val => localStorage.setItem("store", val))

export let teams = [];
export let league_size = 2;

const teamNames = ["Akari","Akito","Hana","Haru","Hinata","Hiroto","Itsuki","Kaito","Kana","Kei","Koharu","Minato","Ren","Riku","Sana","Tsumugi","Yamato"]
let animeList = [];

function setLeagueSize() {
	const oldTeams = teams;
	teams = [];
	for( let i = 0; i < league_size; i++ ) {
		if( i < oldTeams.length ) {
			teams.push( oldTeams[i] );
		}
		else {
			teams.push( {name: teamNames[Math.floor(Math.random() * teamNames.length)], anime: []} );
		}
	}
	saveChanges();
}
function draftAnime() {

}

function saveChanges() {
	$store = JSON.stringify( { teams, animeList } );
}

function loadChanges() {
	const temp = JSON.parse( $store );
	teams = temp.teams;
	animeList = temp.animeList;
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
			<table><tbody>
			{#each team.anime as anime}
				<tr><td>{anime.name}</td><td>{anime.score}</td></tr>
			{/each}
			</tbody></table>
			<button>Make a draft</button>
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