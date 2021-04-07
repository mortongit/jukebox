<script>
	import Playlist from './view/Playlist.svelte';
	// export let name;
	import {setContext} from 'svelte';
	import key from './key';
	setContext('YOUTUBE_API_KEY', key.YOUTUBE_API_KEY);
</script>

<main class="container">
	<h1 class="block">Mini Jukebox</h1>
	<p class="block"><a href="https://svelte.dev">Svelte</a> 공부하기 위해서 만들어 보는 앱</p>
	<div class="columns">
		<div class="column">
			<Playlist></Playlist>
		</div>
	</div>
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>