<script>
import { writable } from "svelte/store";

const store = writable(localStorage.getItem("store") || "");
store.subscribe(val => localStorage.setItem("store", val))

export let targetYear = 2021;
export let targetSeason = "SPRING";
export let currDraftTeam = 0;
export let teams = [];
export let leagueSize = 2;

const avlSeasons = ["WINTER","SPRING","SUMMER","FALL"];
const avlYears = [2021,2022,2023,2024,2025];

let selectedDraftAnime = 0;
let draftVisible = false;

const teamNames = ["Akari","Akito","Hana","Haru","Hinata","Hiroto","Itsuki","Kaito","Kana","Kei","Koharu","Minato","Ren","Riku","Sana","Tsumugi","Yamato"]
let animeList = [];

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
				coverImage { medium }
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

function sortAnimeList() {
	animeList = animeList.sort( (a,b) => { return a["name"] < b["name"] ? -1 : 1 } );
}

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
		animeList = [];
		data.data.Page.media.forEach( element => {
			let name = element["title"]["english"];
			if( name === null ) {
				name = element["title"]["romaji"];
			}
			animeList.push( { id: element["id"], name, averageScore: element["averageScore"], image: element["coverImage"]["medium"] } )
			console.log( element["coverImage"]["large"] );
		} );
		sortAnimeList();
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
	for( let i = 0; i < leagueSize; i++ ) {
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
	const selectedAnime = animeList[selectedDraftAnime];
	let score = parseInt(selectedAnime["averageScore"]);
	if( isNaN(score) ) {
		score = 0;
	}
	teams[currDraftTeam].anime.push( { id: selectedAnime["id"], name: selectedAnime["name"], score } ); // Add to team's anime
	teams[currDraftTeam].totalScore += score;
	animeList.splice( selectedDraftAnime, 1 ); // Remove from anime list
	currDraftTeam += 1; // Go to next team
	if( currDraftTeam >= leagueSize ) {
		currDraftTeam = 0;
	}
	draftVisible = false;
	teams = teams;
}

function saveChanges() {
	$store = JSON.stringify( { leagueSize, currDraftTeam, teams, animeList } );
}

function showDraft() {
	selectedDraftAnime = 0;
	draftVisible = true;
}

function loadChanges() {
	const temp = JSON.parse( $store );
	leagueSize = temp.leagueSize;
	currDraftTeam = temp.currDraftTeam;
	teams = temp.teams;
	animeList = temp.animeList;
}
/* ----- END SCRIPT ----- */
</script>

<nav>
	<h1>Anime Fantasy League</h1>
	<button class="nav-button" on:click={loadChanges}>Load</button>
	<button class="nav-button" on:click={saveChanges}>Save</button>
	<div class="load-bar">
	<label class="load-bar-item">Season:
		<select bind:value={targetSeason}>
			{#each avlSeasons as season}
			<option value={season}>{season}</option>
			{/each}
		</select>
	</label>
	<label class="load-bar-item">Year:
		<select bind:value={targetYear}>
			{#each avlYears as year}
			<option value={year}>{year}</option>
			{/each}
		</select>
	</label>
	<div class="load-bar-item"><button on:click={reloadDataFromAnilist}>Fetch data from AniList</button></div>
	</div>
</nav>

{#if animeList.length > 0}
<main>
	<div class="league-size-container">
	<div>Number of people in your league:</div>
	<input class="league-size" size="2" bind:value={leagueSize} />
	<button on:click={setLeagueSize}>Set</button>
	</div>
	<div class="teams">
		{#each teams as team,teamIndex}
		<div class="team {teamIndex > 0 ? "team-with-border" : ""}">
			<h2><input class="team-name" bind:value={team.name} /></h2>
			<table>
			<thead>
				<tr><th>Anime</th><th>Score</th></tr>
			</thead>
			<tbody>
			{#each team.anime as anime}
				<tr><td class="anime-name">{anime.name}</td><td class="score">{anime.score}</td></tr>
			{/each}
			<tr><td class="total">Total:</td><td class="score total">{team.totalScore}</td></tr>
			</tbody>
			</table>
			{#if teamIndex === currDraftTeam}
				<button class="draft-anime-button" on:click={showDraft}>Draft an anime</button>
			{/if}
		</div>
		{/each}
	</div>
	{#if draftVisible}
		<div class="draft-list">
			<h2>Draft for {teams[currDraftTeam].name}</h2>
			<div class="anime-draft-list">
			{#each animeList as anime,animeIndex}
				<label style="background-image: url({anime["image"]})" class="draft"><input type="radio" name="draft-anime" bind:group={selectedDraftAnime} value={animeIndex} style="vertical-align:top" /> <span style="vertical-align:top">{anime["name"]}</span></label>
			{/each}
			</div>
			<button on:click={draftAnime}>Lock it in!</button>
		</div>
	{/if}
</main>
{/if}

<style>
/* ----- END HTML ----- */

/* Palette: https://www.colourlovers.com/palette/4780148/Palette_for_2021 */
	
	main {
		padding: 1em 0em;
		max-width: 240px;
		margin: 0 auto;
	}

	button {
		background-color: #607AA8;
		color: #FFFBF2;
	}

	h1 {
		text-align: center;
		color: #607AA8;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
		margin: 0.25em;
	}

	h2 {
		text-align: center;
		color: #607AA8;
		font-size: 2em;
	}

	nav {
		border-bottom: 1px solid #8C7643;
	}

	nav .nav-button {
		float: right;
		margin-left: 1em;
	}

	input.league-size {
		text-align: right;
		width: 2em;
		margin-left: 1em;
		margin-right: 1em;
	}

	input.team-name {
		color: #607AA8;
		background-color: transparent;
		border: 0px;
		width: 100%;
		text-align: center;
	}

	table {
		width: 100%;
	}

	td.anime-name {
		font-style: italic;
	}

	td.score {
		text-align: right;
	}

	td.total {
		font-weight: bold;
	}
	tr:nth-child(even) {
		background-color: #f3efe8;
	}

	label.draft {
		flex-grow: 1;
		width: 33%;
		background-repeat: no-repeat;
		height: 142px;
		padding-left: 105px;
	}

	.load-bar {
		display: flex;
		flex-direction: row;
		justify-content: flex-start;
	}

	.load-bar-item {
		margin-right: 1em;
	}

	.league-size-container {
		display: flex;
		flex-direction: row;
		justify-content: flex-start;
	}

	.teams {
		display: flex;
		flex-direction: row;
	}

	.team {
		flex-grow: 1;
		padding: 1em;
		border-top: 1px solid #8C7643;
	}

	.team-with-border {
		border-left: 1px solid #8C7643;
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
		vertical-align: top;
	}

	.anime-draft-list {
		display: flex;
		flex-wrap: wrap;
	}

	.draft-anime-button {
		margin-top: 1em;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>