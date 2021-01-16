<script>
import { writable } from "svelte/store";

const store = writable(localStorage.getItem("store") || "");

store.subscribe(val => localStorage.setItem("store", val))

export let currDraft = 0;
export let teams = [];
export let league_size = 2;

let selectedDraft = 0;
let draftVisible = false;

const teamNames = ["Akari","Akito","Hana","Haru","Hinata","Hiroto","Itsuki","Kaito","Kana","Kei","Koharu","Minato","Ren","Riku","Sana","Tsumugi","Yamato"]
let animeList = [
"So I'm a Spider, So What? [Kumo desu ga, Nani ka?]",
"LAID-BACK CAMP SEASON2 [Yuru Camp SEASON 2]",
"EX-ARM [EX-ARM]",
"Mushoku Tensei: Jobless Reincarnation [Mushoku Tensei: Isekai Ittara Honki Dasu]",
"That Time I Got Reincarnated as a Slime Season 2 [Tensei Shitara Slime Datta Ken 2nd Season]",
"Cells at Work!! [Hataraku Saibou!!]",
"The Promised Neverland Season 2 [Yakusoku no Neverland 2]",
"ICHU [ICHU: HALFWAY THROUGH THE IDOL]",
"The Quintessential Quintuplets 2 [5-Toubun no Hanayome ]",
"Non Non Biyori Nonstop [Non Non Biyori: Nonstop]",
"Attack on Titan Final Season [Shingeki no Kyojin: The Final Season]",
"Bottom-Tier Character Tomozaki [Jaku-Chara Tomozaki-kun]",
"Suppose a Kid from the Last Dungeon Boonies moved to a starter town? [Tatoeba Last Dungeon Mae no Mura no Shounen ga Joban no Machi de Kurasu Youna Monogatari]",
"2.43: Seiin High School Boys Volleyball Team [2.43: Seiin Koukou Danshi Volley-bu]",
"Redo of Healer [Kaifuku Jutsushi no Yarinaoshi]",
"Show by Rock!! Stars!! [SHOW BY ROCK!! STARS!!]",
"Skate-Leading Stars [Skate-LeadingStars]",
"Idoly Pride [IDOLY PRIDE]",
"Dr. STONE: STONE WARS [Dr. STONE: STONE WARS]",
"Kemono Jihen [Kemono Jihen]",
"World Trigger 2nd Season [World Trigger 2]",
"BEASTARS Season 2 [BEASTARS 2]",
"Hortensia SAGA [Hortensia Saga]",
"True Cooking Master Boy Season 2 [Shin Chuuka Ichiban! 2]",
"BACK ARROW [Back Arrow]",
"Log Horizon: Destruction of the Round Table [Log Horizon: Entaku Houkai]",
"Otherside Picnic [Ura Sekai Picnic]",
"The Seven Deadly Sins: Dragon's Judgment [Nanatsu no Taizai: Fundo no Shinpan]",
"WIXOSS DIVA(A)LIVE [WIXOSS DIVA(A)LIVE]",
"Sorcerous Stabber Orphen: Battle of Kimluck [Majutsushi Orphen Hagure Tabi: Kimluck-hen]",
"Cells at Work! CODE BLACK [Hataraku Saibou BLACK]",
"Heaven's Design Team",
"The Hidden Dungeon Only I Can Enter [Ore dake Haireru Kakushi Dungeon]",
"LBX Girls [Soukou Musume Senki]",
"Re:ZERO -Starting Life in Another World- Season 2 Part 2 [Re:Zero kara Hajimeru Isekai Seikatsu 2nd Season Part 2]",
"Dr. Ramune -Mysterious Disease Specialist- [Kai Byoui Ramune]",
"Scar on the Praeter [Praeter no Kizu]",
"Horimiya [Horimiya]",
"Gekidol [Gekidol]",
"SK8 the Infinity [SK]",
"Umamusume: Pretty Derby Season 2 [Uma Musume: Pretty Derby Season 2]",
"WONDER EGG PRIORITY [Wonder Egg Priority]"
];

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
	console.log( selectedDraft );
	const selectedAnime = animeList[selectedDraft];
	console.log( selectedAnime );
	teams[currDraft].anime.push( { name: selectedAnime, score: 0 } ); // Add to user's anime
	animeList.splice( selectedDraft, 1 ); // Remove from anime list
	currDraft += 1; // Go to next user
	if( currDraft >= league_size ) {
		currDraft = 0;
	}
	draftVisible = false;
	teams = teams;
}

function saveChanges() {
	$store = JSON.stringify( { teams, animeList } );
}

function showDraft() {
	draftVisible = true;	
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
		{#each teams as team,teamIndex}
		<div class="team">
			<h2><input bind:value={team.name} /></h2>
			<table><tbody>
			{#each team.anime as anime}
				<tr><td>{anime.name}</td><td>{anime.score}</td></tr>
			{/each}
			</tbody></table>
			{#if teamIndex == currDraft}
				<button on:click={showDraft}>Make a draft</button>
			{/if}
		</div>
		{/each}
	</div>
	{#if draftVisible}
		<div class="draft-list">
			<h2>Draft for {teams[currDraft].name}</h2>
			{#each animeList as anime,animeIndex}
				<label><input type="radio" name="draft-anime" bind:group={selectedDraft} value={animeIndex} /> {anime}</label>
			{/each}
			<button on:click={draftAnime}>Save this draft</button>
		</div>
	{/if}
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

	.draft-list {
		position: absolute;
		left: 5em;
		top: 5em;
		right: 5em;
		bottom: 5em;
		overflow-y: scroll;
		background-color: #FFFBF2;
		border: 1px solid #1D487D;
		border-radius: 1em;
		padding: 1em;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>