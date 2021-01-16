<script>
import { writable } from "svelte/store";

const store = writable(localStorage.getItem("store") || "");

store.subscribe(val => localStorage.setItem("store", val))

export let targetYear = 2021;
export let targetSeason = "SPRING";
export let currDraft = 0;
export let teams = [];
export let league_size = 2;

let selectedDraft = 0;
let draftVisible = false;

const teamNames = ["Akari","Akito","Hana","Haru","Hinata","Hiroto","Itsuki","Kaito","Kana","Kei","Koharu","Minato","Ren","Riku","Sana","Tsumugi","Yamato"]
let animeList = [
"2.43: Seiin High School Boys Volleyball Team [2.43: Seiin Koukou Danshi Volley-bu]",
"Attack on Titan Final Season [Shingeki no Kyojin: The Final Season]",
"BACK ARROW [Back Arrow]",
"BEASTARS Season 2 [BEASTARS 2]",
"Bottom-Tier Character Tomozaki [Jaku-Chara Tomozaki-kun]",
"Cells at Work!! [Hataraku Saibou!!]",
"Dr. STONE: STONE WARS [Dr. STONE: STONE WARS]",
"EX-ARM [EX-ARM]",
"Hortensia SAGA [Hortensia Saga]",
"ICHU [ICHU: HALFWAY THROUGH THE IDOL]",
"Idoly Pride [IDOLY PRIDE]",
"Kemono Jihen [Kemono Jihen]",
"LAID-BACK CAMP SEASON2 [Yuru Camp SEASON 2]",
"Mushoku Tensei: Jobless Reincarnation [Mushoku Tensei: Isekai Ittara Honki Dasu]",
"Non Non Biyori Nonstop [Non Non Biyori: Nonstop]",
"Redo of Healer [Kaifuku Jutsushi no Yarinaoshi]",
"Show by Rock!! Stars!! [SHOW BY ROCK!! STARS!!]",
"Skate-Leading Stars [Skate-LeadingStars]",
"So I'm a Spider, So What? [Kumo desu ga, Nani ka?]",
"Suppose a Kid from the Last Dungeon Boonies moved to a starter town? [Tatoeba Last Dungeon Mae no Mura no Shounen ga Joban no Machi de Kurasu Youna Monogatari]",
"That Time I Got Reincarnated as a Slime Season 2 [Tensei Shitara Slime Datta Ken 2nd Season]",
"The Promised Neverland Season 2 [Yakusoku no Neverland 2]",
"The Quintessential Quintuplets 2 [5-Toubun no Hanayome ]",
"World Trigger 2nd Season [World Trigger 2]",
"True Cooking Master Boy Season 2 [Shin Chuuka Ichiban! 2]",
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

const aniListQuery = JSON.stringify({
	query: `{
		Page {
			pageInfo {
				total
				currentPage
				lastPage
				hasNextPage
				perPage
			}
			media (season: WINTER, seasonYear: 2021, type: ANIME, format: TV) {
				id
				startDate { year, month, day }
				format
				title {
					romaji
					english
				}
				source
				genres
				episodes
				duration
				averageScore
				meanScore
				popularity
				}
			}
		}
	`
});

const options = {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Content-Length': aniListQuery.length,
    'User-Agent': 'Node',
  },
  body: aniListQuery
};

function refreshAnilistData() {
	fetch('https://graphql.anilist.co', getOptionsForSeason(targetYear,targetSeason)).then( r => r.json() ).then( data => {
		console.log( "Success" );
		console.log( data.data.Page.media );
	}).catch( err => {
		console.error( "No good." );
	});
}

function reloadDataFromAnilist() {
	fetch('https://graphql.anilist.co', getOptionsForSeason(targetYear,targetSeason)).then( r => r.json() ).then( data => {
		console.log( "Success" );
		//console.log( data.data.Page.media );
		animeList = [];
		data.data.Page.media.forEach( element => {
			let name = element["title"]["english"];
			if( name === null ) {
				name = element["title"]["romaji"];
			}
			animeList.push( { id: element["id"], name, averageScore: element["averageScore"] } )
		} );
	}).catch( err => {
		console.error( "No good." );
	});
}

function getOptionsForSeason( year, season ) {
	let myOptions = options;
	myOptions.body = myOptions.body.replace( "2021", year );
	myOptions.body = myOptions.body.replace( "WINTER", season );
	return myOptions;
}

function setLeagueSize() {
	const oldTeams = teams;
	teams = [];
	for( let i = 0; i < league_size; i++ ) {
		if( i < oldTeams.length ) {
			teams.push( oldTeams[i] );
		}
		else {
			teams.push( {name: teamNames[Math.floor(Math.random() * teamNames.length)], anime: [], totalScore: 0} );
		}
	}
	saveChanges();
}
function draftAnime() {
	const selectedAnime = animeList[selectedDraft];
	let score = parseInt(selectedAnime["averageScore"]);
	if( isNaN(score) ) {
		score = 0;
	}
	teams[currDraft].anime.push( { id: selectedAnime["id"], name: selectedAnime["name"], score } ); // Add to user's anime
	teams[currDraft].totalScore += score;
	animeList.splice( selectedDraft, 1 ); // Remove from anime list
	currDraft += 1; // Go to next user
	if( currDraft >= league_size ) {
		currDraft = 0;
	}
	draftVisible = false;
	teams = teams;
}

function calcScore( team ) {
	console.log( teams[team].anime );
	//return teams[team].anime.reduce( (a,c) => a + c.score );
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
	<button on:click={reloadDataFromAnilist}>Fetch data from AniList</button>
</nav>

<main>
	<p>Number of people in your league:</p>
	<input name="league_size" bind:value={league_size} />
	<button on:click={setLeagueSize}>Set</button>
	<div class="teams">
		{#each teams as team,teamIndex}
		<div class="team">
			<h2><input bind:value={team.name} /></h2>
			<table>
			<thead>
				<tr><th>Anime Title</th><th>Score</th></tr>
			</thead>
			<tbody>
			{#each team.anime as anime}
				<tr><td>{anime.name}</td><td>{anime.score}</td></tr>
			{/each}
			<tr><td><em>Total:</em></td><td>{team.totalScore}</td></tr>
			</tbody>
			</table>
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
				<label><input type="radio" name="draft-anime" bind:group={selectedDraft} value={animeIndex} /> {anime["name"]}</label>
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