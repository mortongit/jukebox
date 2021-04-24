<div class="column is-centered">
    <div on:click="{signIn}" class="is-clickable" id="imgbtn" />
</div>

<script>
    import { navigate } from "svelte-routing"; //

    var provider = new firebase.auth.GoogleAuthProvider();
    function signIn() {
        firebase.auth()
        .signInWithPopup(provider)
        .then((result) => {
            console.log(result);
            // var credential = result.credential; //@type {firebase.auth.OAuthCredential}
            // var token = credential.accessToken; //Google API 사용을 위한 Google Access Token
            // var user = result.user; //로그인한 사용자 정보
            console.log(result.user.email,result.user.displayName);
            console.log('사용자UID',result.user.uid);
            navigate("/home");  //navigate("/home", { replace: true }); 
        }).catch((error) => {
            console.log(error); //error.code, error.message, error.email, error.credential
            alert('로그인 실패');
        });
    }



</script>
<style>
    #imgbtn {
        /* display: block; */
        /* -moz-box-sizing: border-box; */
        /* box-sizing: border-box; */
        width: 382px; /* Width of new image */
        height: 92px; /* Height of new image */
        background: url(../btn_google_signin_light_normal_web@2x.png) no-repeat;
        margin: auto;
    }
    #imgbtn:hover {
        background: url(../btn_google_signin_light_focus_web@2x.png) no-repeat;
    }
    #imgbtn:active {
        background: url(../btn_google_signin_light_pressed_web@2x.png) no-repeat;
    }
    

</style>