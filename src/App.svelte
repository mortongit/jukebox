<script>
	import { Router, Link, Route, navigate } from 'svelte-routing';
	import Playlist from './view/Playlist.svelte';
	import Login from './view/Login.svelte';
	import UserHome from './view/UserHome.svelte';
	export let url = "";

	// export let name;
	import {setContext} from 'svelte';
	import key from './key';
	setContext('YOUTUBE_API_KEY', key.YOUTUBE_API_KEY);
	
	let isYouTubeIframeAPIReady = false;
	window.onYouTubeIframeAPIReady = ()=>isYouTubeIframeAPIReady=true;
	
	var tag = document.createElement("script");
	tag.src = "https://www.youtube.com/iframe_api";
	var firstScriptTag = document.getElementsByTagName("script")[0];
	firstScriptTag.parentNode.insertBefore(tag, firstScriptTag);
	
</script>

<main class="container">
	<!-- 유튜브 IFRAME PLAYER API -->
	<!-- <script src="https://www.youtube.com/iframe_api"></script> -->

	{#if isYouTubeIframeAPIReady}
	<h1 class="block">Mini Jukebox</h1>
	<!-- <p class="block"><a href="https://svelte.dev">Svelte</a> 공부할겸 만들어 보는 앱</p> -->
	<Login />
	<div class="columns">
		<div class="column">
			<Router url="{url}">
				<!-- <nav>
				  <Link to="/">Login</Link>
				  <Link to="home">Home</Link>
				  <Link to="play/favorites">Playlist</Link>
				</nav> -->
				<div>
				  <!-- <Route path="play/:pid" let:params ><Playlist pid="{params.pid}" /></Route> -->
				  <Route path="play/:pid" component="{Playlist}" ></Route>
				  <Route path="home" ><UserHome /></Route>
				  <!-- <Route path="/"></Route> -->
				</div>
			</Router>
		</div>
	</div>
	{/if}
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