<div class="columns">
    <div class="column">
      <nav class="panel is-info">
        <div class="panel-heading">
            <span class="tag is-primary is-light is-medium">PRIVATE</span>
        </div>
        {#each docList as item,idx (item.id) }
        <div class="panel-block" >
          <div class="columns is-centered is-vcentered">
            <div class="column is-narrow-tablet">
              <span class="panel-icon fa-2x">
                <i class="fas fa-list" aria-hidden="true"></i>
              </span>
            </div>
            <div class="column">
              <Link to="play/{item.id}" class="title is-6">{item.data().title}</Link>
            </div>
            <div class="column is-narrow-tablet">
              <button on:click="{ev=>delPlaylist(item.id)}" class="delete is-large"></button>
            </div>
          </div>
        </div>
        {/each}
        <div class="panel-block">
            <div class="control">
                <div class="field is-grouped">
                <div class="control is-expanded">
                    <input type="search" bind:value="{ptitle}" placeholder="플레이리스트 이름" class="input" />
                </div>
                <div class="control">
                    <button type="button" on:click="{addPlaylist}" class="button is-info">
                    <span class="icon is-small">
                        <i class="fas fa-plus" aria-hidden="true"></i>
                    </span>
                    </button>
                </div>
                </div>
            </div>
        </div>
      </nav>
    </div>
    <div  class="column">
      <nav class="panel is-info">
        <div class="panel-heading"><span class="tag is-primary is-light is-medium">PUBLIC</span></div>
        <div class="panel-block">
          <div class="control">
            <div class="field is-grouped">
              <div class="control is-expanded has-icons-left">
                <input type="search" placeholder="검색어" class="input" disabled />
                <span class="icon is-left">
                  <i class="fas fa-search" aria-hidden="true"></i>
                </span>
              </div>
              <div class="control">
                <button type="button" class="button is-info">
                  <span class="icon is-small">
                    <i class="fas fa-search" aria-hidden="true"></i>
                  </span>
                </button>
              </div>
            </div>
          </div>
        </div>
      </nav>
    </div>
  </div>

<script>
  import { Router, Link, Route, navigate } from 'svelte-routing';
  var db = firebase.firestore();

  let ptitle = ''; //플레이리스트 이름 입력
  // let playlists = [];
  let docList = []; //플레이리스트 목록

  function readPlaylists(params) {
    // db.collection("playlists").where("uid", "==", firebase.auth().currentUser.uid).get()
    db.collection("playlists").where("uid", "==", firebase.auth().currentUser.uid).orderBy("title").get()
    .then((querySnapshot) => {
      docList.splice(0); //배열 비우기
      querySnapshot.forEach((doc) => docList.push(doc)); //플레이리스트 목록에 추가
      docList = docList; //반응성을 위해서 대입
      // console.log("getting playlists: ", playlists);
    }).catch((error) => {
      console.log("Error getting playlists:", error);
    });
  }

  readPlaylists(); //최초 플레이리스트 목록 조회

  function addPlaylist() {
      let docRef = db.collection("playlists").doc(); //문서ID 자동생성
      docRef.set({
        // id: docRef.id, //플레이리스트 아이디
        title: ptitle, //플레이리스트 이름
        tracks: [],  //플레이리스트 동영상 목록
        uid: firebase.auth().currentUser.uid, //플레이리스트 소유자
        timestamp: firebase.firestore.FieldValue.serverTimestamp() //등록 시간 저장
      })
      .then(() => {
          console.log("Document successfully written!");
          readPlaylists(); //추가 후 플레이리스트 목록 다시 조회
      })
      .catch((error) => {
          console.error("Error writing document: ", error);
      });
  }

  function delPlaylist(docId) {
    db.collection("playlists").doc(docId).delete().then(() => {
        console.log("Document successfully deleted!");
        readPlaylists(); //삭제 후 플레이리스트 목록 다시 조회
    }).catch((error) => {
        console.error("Error removing document: ", error);
    });
  }

  // import { onMount, onDestroy } from 'svelte';
  // onDestroy(() => {
  //     console.log('onDestroy');
  //     unsubscribePlaylists(); // 디스트로이 라이프 사이클에서 업데이트 수신 대기 취소
  // });

  // firebase.auth().onAuthStateChanged(function(user) {
  //     if (user) {
  //         // User is signed in.
  //     } else {
  //         // No user is signed in.
  //     }
  // });
</script>

