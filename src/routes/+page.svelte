<script>
	import Leaderboard from '$lib/components/Leaderboard.svelte'
	import TodaysTopPerformers from '$lib/components/TodaysTopPerformers.svelte'
	import ScoringBreakdown from '$lib/components/ScoringBreakdown/index.svelte'
	import TopScorersOverall from '$lib/components/TopScorersOverall.svelte'
	import TeamBreakdown from '$lib/components/TeamBreakdown.svelte'
	import DraftAnalysis from '$lib/components/DraftAnalysis.svelte'

	export let data
	const { games, gameStats, participantScores } = data
</script>

<svelte:head>
	<title>Scoring Pool</title>
</svelte:head>

<h1>NBA Playoffs 2023 Scoring Pool</h1>

<p>
	These stats should update every 10 minutes or so. You need to refresh your page for the stats to
	update.
</p>

<Leaderboard {participantScores} />
<TodaysTopPerformers {gameStats} {games} />
<ScoringBreakdown {gameStats} />
<TopScorersOverall {gameStats} />
<TeamBreakdown {gameStats} />
<DraftAnalysis {gameStats} />
