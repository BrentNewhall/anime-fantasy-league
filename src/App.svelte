<script>
import { writable } from "svelte/store";

// import * as AWS from 'aws-sdk';
// export const dbConfig = {
//   region: 'us-east-1',
//   endpoint: 'dynamodb.us-east-1.amazonaws.com',
//   accessKeyId: 'keyid',
//   secretAccessKey: 'accesskey',
// }
// export const tableName = "mini-db-test";
// AWS.config.update( dbConfig );
// const dynamodb = new AWS.DynamoDB();
// const docClient = new AWS.DynamoDB.DocumentClient();
// const params = {
//     TableName: tableName,
// }
// docClient.scan( params, (err, data) => {
// 	if( data !== null ) {
// 		const items = data.Items;
// 		console.log( items );
// 	}
// });

// Use LocalStorage
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
let fetchDataVisible = true;
let draftVisible = false;
let disqualifyVisible = false;
let nextTeamRandom = false;
let nextTeamRandomAvailable = false;
let teamsChosen = [];

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
		} );
		sortAnimeList();
		fetchDataVisible = false;
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

function nameTaken( teams, name ) {
	return (teams.filter( team => team["name"] === name )).length > 0;
}

function getNewName(teamNames, teams) {
	let name = teamNames[Math.floor(Math.random() * teamNames.length)];
	while( nameTaken( teams, name ) ) {
		name = teamNames[Math.floor(Math.random() * teamNames.length)];
	}
	return name;
}

function setLeagueSize() {
	const oldTeams = teams;
	teams = [];
	for( let i = 0; i < leagueSize; i++ ) {
		if( i < oldTeams.length ) {
			teams.push( oldTeams[i] );
		}
		else {
			teams.push( {name: getNewName( teamNames, teams ), anime: [], totalScore: 0} );
		}
	}
	if( teams.length > 2 ) {
		nextTeamRandomAvailable = true;
	}
	else {
		nextTeamRandomAvailable = false;
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
	if( nextTeamRandom ) {
		saveTeamChosen();
		nextRandomTeam();
	}
	else {
		currDraftTeam += 1; // Go to next team
		if( currDraftTeam >= leagueSize ) {
			currDraftTeam = 0;
		}
	}
	draftVisible = false;
	teams = teams;
}

function saveTeamChosen() {
	teamsChosen.push( currDraftTeam );
	if( teamsChosen.length > teams.length * 2 ) {
		teamsChosen.shift();
	}
	console.log( teamsChosen );
}

function nextRandomTeam() {
	let forceTeam = false;
	for( let team = 0; team < teams.length; team++ ) {
		if( ! teamsChosen.includes( team ) ) {
			currDraftTeam = team;
			forceTeam = true;
			break;
		}
	}
	if( ! forceTeam ) {
		let t = currDraftTeam;
		while( t === currDraftTeam ) {
			t = Math.floor(Math.random() * teams.length);
		}
		currDraftTeam = t;
	}
}

function saveChanges() {
	$store = JSON.stringify( { leagueSize, currDraftTeam, teams, animeList } );
}

function showFetchDataPanel() {
	fetchDataVisible = true;
}

function showDraft() {
	selectedDraftAnime = 0;
	draftVisible = true;
}

function showDisqualify() {
	disqualifyVisible = true;
}

function doneDisqualifying() {
	for( let i = animeList.length - 1; i >= 0; i-- ) {
		if( document.getElementById("disqualify-anime-" + i).checked ) {
			animeList.splice( i, 1 );
		}
	}
	disqualifyVisible = false;
}

function toggleNextTeamRandom() {
	nextTeamRandom = document.getElementById("next-team-random").checked;
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
	<button class="nav-button" on:click={saveChanges}>Save</button>
	<button class="nav-button" on:click={loadChanges}>Load</button>
	<button class="nav-button" on:click={showFetchDataPanel}>Fetch</button>
	<h1>Anime Fantasy League</h1>
</nav>

{#if animeList.length > 0}
<main>
	<div class="league-size-container">
	<div>Number of people in your league:</div>
	<input class="league-size" size="2" bind:value={leagueSize} />
	<button on:click={setLeagueSize}>Set</button>
	<label class="next-team-random">
		<input type="checkbox" disabled={nextTeamRandomAvailable ? "" : "disabled"} id="next-team-random" on:click={toggleNextTeamRandom} /> <span class={nextTeamRandomAvailable ? "" : "next-team-random-disabled"}>Randomize draft order</span>
	</label>
	<button on:click={showDisqualify} class="btn-disqualify">Disqualify</button>
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
	{#if disqualifyVisible}
		<div class="draft-list">
			<h2>Disqualify These Anime</h2>
			<div class="anime-draft-list">
			{#each animeList as anime,animeIndex}
				<label style="background-image: url({anime["image"]})" class="draft"><input type="checkbox" id="disqualify-anime-{animeIndex}" style="vertical-align:top" /> <span style="vertical-align:top">{anime["name"]}</span></label>
			{/each}
			</div>
			<button on:click={doneDisqualifying}>Done</button>
		</div>
	{/if}
</main>
{/if}

	{#if fetchDataVisible}
		<div class="load-screen">
		<h2>Load Data For When?</h2>
		<label>Season:
			<select bind:value={targetSeason}>
				{#each avlSeasons as season}
				<option value={season}>{season}</option>
				{/each}
			</select>
		</label>
		<label>Year:
			<select bind:value={targetYear}>
				{#each avlYears as year}
				<option value={year}>{year}</option>
				{/each}
			</select>
		</label>
		<button on:click={reloadDataFromAnilist}>Fetch data from AniList</button>
		</div>
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
		text-align: left;
		text-transform: capitalize;
		font-size: 3em;
		font-weight: 100;
		margin: 0.25em;
	}

	h2 {
		text-align: center;
		color: #607AA8;
		font-size: 2em;
	}

	nav {
		color: white;
		background-color: #1D487D;
		border-bottom: 1px solid #8C7643;
		border-top-left-radius: 1em;
		border-top-right-radius: 1em;
	}

	nav .nav-button {
		float: right;
		margin-left: 0.5em;
		margin-top: 1em;
		margin-right: 1em;
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
		width: 25%;
		background-repeat: no-repeat;
		height: 142px;
		padding-left: 105px;
	}

	label.next-team-random {
		margin-left: 1em;
	}

	.next-team-random-disabled {
		color: gray;
	}

	.load-bar {
		display: flex;
		flex-direction: row;
		justify-content: flex-start;
	}

	.load-bar-item {
		margin-right: 1em;
	}

	.btn-disqualify {
		margin-left: 2em;
		color: white;
		background-color: #8C7643;
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

	.load-screen {
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
		text-align: center;
	}

	.load-screen label {
		text-align: center;
	}

	.load-screen button {
		text-align: center;
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