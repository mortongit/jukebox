<div class="column is-centered">
	<p class="block has-text-centered">
    {#if loginUser}
        <!-- 로그인한 경우 로그아웃 버튼 출력 -->
        <button on:click="{signOut}" class="button">
			<span class="icon"><i class="fas fa-sign-out-alt"></i></span>
			<span>Sign Out</span>
		</button>
    {:else if loginable}
        <!-- 로그인하지 않은 경우 로그인 준비가 되면 로그인 버튼 출력 -->
        <button on:click="{signIn}" class="button">
            <span class="icon">
                <i class="fab fa-google"></i>
            </span>
            <span>Sign in with Google</span>
        </button>
    {/if}
    </p>
</div>

<script>
    import { navigate } from "svelte-routing"; //

    let loginUser = null; //현재 로그인한 사용자
    let loginable = false; //로그인 가능 상태 여부
  
    firebase.auth().setPersistence(firebase.auth.Auth.Persistence.SESSION) //브라우저/탭 종료 전까지 로그인 유지
    .then(() => { //설정 완료시 
        loginable = true; //로그인 가능
        firebase.auth().onAuthStateChanged(function(user) { //최초 설정시와 로그인 상태 변경시 자동 실행
            if (user) {
                console.log('User is signed in.');
                loginUser = user; //현재 로그인한 사용자 정보 저장
                navigate('/home');  //로그인을 성공시 또는 이미 로그인되어 있는 상태인 경우, '/home' 경로로 이동
            } else {
                console.log('No user is signed in.');
                loginUser = null; //현재 로그인한 사용자 정보 삭제
                navigate('/'); //다행히 현재 경로가 이미 '/'이면 이동하지 않는 듯
            }
        });
    })
    .catch((error) => {
        console.log(error.code, error.message);
    });

    var provider = new firebase.auth.GoogleAuthProvider();
    function signIn() {
        firebase.auth().signInWithPopup(provider)
        .then((result) => {
            console.log(result);
            // var credential = result.credential; //@type {firebase.auth.OAuthCredential}
            // var token = credential.accessToken; //Google API 사용을 위한 Google Access Token
            // var user = result.user; //로그인한 사용자 정보
            console.log(result.user.email,result.user.displayName);
            console.log('사용자UID',result.user.uid);
            // onAuthStateChanged 리스너가 실행되어 /home 화면으로 이동
            // navigate("/home");  //navigate("/home", { replace: true }); 
        }).catch((error) => {
            console.log(error); //error.code, error.message, error.email, error.credential
            alert('로그인 실패');
        });
    }

    function signOut(params) {
		if (confirm('로그 아웃 할까요?')) {
			firebase.auth().signOut().then(() => { //로그 아웃 성공시
                console.log('User is signed out.');
                // onAuthStateChanged 리스너가 실행되어 / 화면으로 이동
				// navigate("/");  //navigate("/", { replace: true }); 
			}).catch((error) => { // An error happened.
				alert('로그아웃 실패');
			});
		}
	}
</script>
<style>

</style>