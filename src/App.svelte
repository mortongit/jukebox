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

	function signout(params) {
		if (confirm('로그 아웃 할까요?')) {
			firebase.auth().signOut().then(() => { // Sign-out successful.
				navigate("/");  //navigate("/", { replace: true }); 
			}).catch((error) => { // An error happened.
				alert('로그아웃 실패');
			});
		}
	}
</script>

<main class="container">
	<h1 class="block">Mini Jukebox</h1>
	<p class="block"><a href="https://svelte.dev">Svelte</a> 공부할겸 만들어 보는 앱</p>
	<!-- 반응성을 구현하기 위하여 store 사용 고려 -->
	{#if window.firebase.auth().currentUser !== null }
	<p class="block has-text-centered">
		<button on:click="{signout}" class="button">
			<span class="icon"><i class="fas fa-sign-out-alt"></i></span>
			<span>Sign Out</span>
		</button>
	</p>
	{/if}
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
				  <Route path="/"><Login /></Route>
				</div>
			</Router>
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