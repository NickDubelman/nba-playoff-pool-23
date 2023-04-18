<script>
	import participants from '$lib/participants'
	import { fullName, remainingTeams, nameDifferences, teamNameDifferences } from '$lib/utils'

	export let data
	const { games, gameStats, participantScores } = data

	const playerStats = {} // a map from player name to corresponding stats

	let teamProjectedGames = {
		DEN: 10,
		MEM: 6,
		SAC: 5,
		PHO: 23,
		LAC: 4,
		GSW: 12,
		LAL: 17,
		MIN: 3,
		MIL: 23,
		BOS: 17,
		PHI: 11,
		CLE: 12,
		NYK: 6,
		BRK: 4,
		MIA: 4,
		ATL: 3
	}

	let playerProjectedPPG = {}
	let playerTeams = {}

	gameStats.forEach(({ player, team, pts }) => {
		let name = fullName(player)
		if (nameDifferences[name]) {
			name = nameDifferences[name]
		}

		// First time we see a player, initialize an entry in the map for them
		if (!playerStats[name]) {
			playerStats[name] = {
				team: team.abbreviation,
				pts: 0,
				games: 0
			}
		}

		playerStats[name].pts += pts
		playerStats[name].games++
	})

	function round(num) {
		return Math.round(num * 10) / 10
	}

	function getPlayerPoints(playerName) {
		if (nameDifferences[playerName]) {
			playerName = nameDifferences[playerName]
		}

		return playerStats[playerName]?.pts || 0
	}

	function getPlayerPPG(playerName) {
		if (nameDifferences[playerName]) {
			playerName = nameDifferences[playerName]
		}

		const player = playerStats[playerName]
		if (!player) {
			return 0
		}

		return round((player.pts || 0) / (player.games || 1))
	}

	function getParticipant(participantName) {
		return participantScores.find((p) => p.name === participantName)
	}

	$: getTeamProjectedGames = (team) => {
		if (teamNameDifferences[team]) {
			team = teamNameDifferences[team]
		}

		return teamProjectedGames[team] || 0
	}

	$: getPlayerProjectedPPG = (playerName) => {
		return playerProjectedPPG[playerName] || 0
	}

	$: getPlayerTeam = (playerName) => {
		if (nameDifferences[playerName]) {
			playerName = nameDifferences[playerName]
		}

		return playerStats[playerName]?.team || playerTeams[playerName] || ''
	}
</script>

<svelte:head>
	<title>Scoring Pool - Simulator</title>
</svelte:head>

<h1>Simulator</h1>

<p>proj points = total points + proj ppg * proj remaining games</p>

<!-- show a table where each row is a remaining team, user enters number of projected remaining games -->
<table>
	<thead>
		<tr>
			<th>Team</th>
			<th>Proj Remaining Games</th>
		</tr>
	</thead>

	<tbody>
		{#each remainingTeams as team}
			<tr>
				<td>{team}</td>
				<td>
					<input
						type="number"
						min="0"
						value={getTeamProjectedGames(team)}
						on:change={(e) => (teamProjectedGames[team] = e.target.value)}
					/>
				</td>
			</tr>
		{/each}
	</tbody>
</table>

<!-- For each participant we will show a bordered table -->
{#each Object.entries(participants) as [participant, players]}
	<h2>{participant}</h2>
	<table>
		<thead>
			<tr>
				<th>Player</th>
				<th>Total Points</th>
				<th>PPG</th>
				<th>Proj PPG</th>
				<th>Proj Points</th>
			</tr>
		</thead>

		<tbody>
			{#each players as player}
				<tr>
					<td>{player}</td>
					<td>{getPlayerPoints(player)}</td>
					<td>
						<!-- average number of points for the player, rounded to 1 decimal -->
						{getPlayerPPG(player)}
					</td>

					<td>
						<input
							type="number"
							min="0"
							value={playerProjectedPPG[player] || 0}
							on:change={(e) => (playerProjectedPPG[player] = e.target.value)}
						/>
					</td>

					<td>
						{getPlayerPoints(player) +
							getPlayerProjectedPPG(player) * getTeamProjectedGames(getPlayerTeam(player))}
					</td>
				</tr>
			{/each}

			<!-- show a row for totals -->
			<tr>
				<td>Totals</td>
				<td>{getParticipant(participant).points}</td>
				<td>
					<!-- average number of points for the participant, rounded to 1 decimal -->
					{round(getParticipant(participant).points / getParticipant(participant).gamesPlayed)}
				</td>
				<td>--</td>
				<td>
					<!-- add up the projected points for each player on the participants team -->
					{players.reduce(
						(total, player) =>
							total +
							getPlayerPoints(player) +
							getPlayerProjectedPPG(player) * getTeamProjectedGames(getPlayerTeam(player)),
						0
					)}
				</td>
			</tr>
		</tbody>
	</table>
{/each}

<style>
	/* basic bordered table */
	table {
		border-collapse: collapse;
	}

	table,
	th,
	td {
		border: 1px solid grey;
	}
</style>
