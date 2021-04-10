<div bind:this="{rootElm}" class="columns">
  <div class="column">
    <nav class="panel is-info">
      <div class="panel-heading">
        <span class="tag is-primary is-light is-medium">{songList.length}</span>
        <span class="tag is-primary is-light is-medium">{new Date(songList.reduce((pv,cv)=>pv+cv.duration,0)*1000).toUTCString().split(' ')[4]}</span>
      </div>
      <div class="sortable">
      {#each songList as item,idx (item) }
        <!-- svelte-ignore a11y-missing-attribute -->
        <div class="panel-block {songIdx===idx?'is-active':''}" >
          <div class="columns is-centered is-vcentered">
            <div class="column is-narrow-tablet">
              <span class="panel-icon fa-2x">
                <i on:click="{ev=>clickSong(idx)}" class="fas fa-compact-disc {songIdx===idx?'fa-spin':''}" aria-hidden="true"></i>
              </span>
            </div>
            <div class="column handle">
              <p class="title is-6">{item.title}</p>
              <p class="subtitle is-6">
                {new Date(item.duration*1000).toUTCString().split(' ')[4]} 
                <!-- ( {item.start} ~ {item.end} )  -->
              </p>
            </div>
            <div class="column is-narrow-tablet">
              <button on:click="{ev=>delSong(idx)}" class="delete is-large"></button>
            </div>
          </div>
        </div>
      {/each}
      </div>
      <div class="panel-block">
        <input type="range" min="{seekMin}" max="{seekMax}" bind:value="{seekVal}" on:change={seekChange} class="form-range" >
      </div>
      <div class="panel-block">
        <button type="button" on:click="{startVideo}" class="button is-info is-fullwidth {songBuffering?'is-loading':''}"><i class="fas {songPlaying?'fa-pause':'fa-play'} {songPlaying?'bi-pause':'bi-play'}"></i></button>
      </div>
    </nav>

    <div class="field">
      <input id="switchRoundedSuccess" type="checkbox" name="switchRoundedSuccess" class="switch is-rounded is-success" bind:checked="{debug}" on:change="{changeDebug}">
      <label for="switchRoundedSuccess">DEBUG</label>
    </div>

    <!-- <iframe> (과 유튜브 플레이어)로 대체할 <div> 엘리먼트 -->
    <!-- <iframe id="player" bind:this="{playerElm}" width="100" height="100" src="https://www.youtube.com/embed/DN20QSP3CqI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> -->
    <div id="player"></div>
  </div>
  <div  class="column">
    <nav class="panel is-info">
      <div class="panel-heading">검색</div>
      <div class="panel-block">
        <!-- <div class="control">
          <div class="field is-grouped"> -->
            <div class="control is-expanded has-icons-left">
              <input type="search" bind:value="{searchWord}" on:keydown="{e=>{if(e.key==='Enter')searchSong()}}" placeholder="검색어 또는 주소" class="input" />
              <span class="icon is-left">
                <i class="fas fa-search" aria-hidden="true"></i>
              </span>
            </div>
            <!-- <div class="control">
              <button type="button" on:click="{searchSong}" class="button is-info">
                <span class="icon is-small">
                  <i class="fas fa-search" aria-hidden="true"></i>
                </span>
              </button>
            </div>
          </div>
        </div> -->
      </div>
      {#each searchList as item,idx (item.id) }
        <!-- svelte-ignore a11y-missing-attribute -->
        <div class="panel-block" >
          <div class="columns is-centered is-vcentered">
            <div class="column is-narrow-tablet">
              <span class="panel-icon fa-2x">
                <i on:click="{ev=>addToSongList(item)}" class="fas fa-plus" aria-hidden="true"></i>
              </span>
            </div>
            <div class="column">
              <p class="title is-6">{item.title}</p>
              <p class="subtitle is-6"><span class="tag {item.type==='video'?'is-success':'is-warning'}">{item.type}</span></p>
            </div>
            <div class="column is-narrow-tablet has-text-centered">
              <figure class="image is-64x64 is-inline-block">
                <img src="{item.img}">
              </figure>
            </div>
          </div>
        </div>
      {/each}
    </nav>
  </div>
</div>

<script>
  // 동영상 보이기 감추기
  let debug = true;
  function changeDebug(ev) {
    if (debug) player.setSize(320, 180);
    else player.setSize(0, 0);
  }
  // https://youtu.be/DN20QSP3CqI
  // https://youtu.be/Sl5I4I0ZDrY
  // https://youtu.be/lpwG8f9nt4s
  // https://youtu.be/LSVgeVWxXV8
  // https://youtu.be/eWuRCEIr5jQ
  // https://www.youtube.com/watch?v=DN20QSP3CqI
  // https://www.youtube.com/playlist?list=PLgt61A4grz6hUuQ7vvLXCYFk_gyvXqx23

  // TODO
  // 1.검색 결과 페이지 처리 (현재는 검색결과를 20개까지만 지원)
  // 2.재생목록은 50곡까지 지원 (재생목록 추가시 검색결과로 표시하고 페이지 처리를 구현하여 해결 가능)
  // 3.동영상 재생시작 및 종료시간 지정 기능 구현
  // 4.가끔씩 드래그앤드롭 재생순서 조정이 작동하지 않는 문제 해결
  // 5.컴포넌트 분리
  // 6.유튜브 재생목록 임포트시 비동기 조회로 인해 순서가 유지되지 않음

  import { getContext } from 'svelte'
  const youtubeApiKey = getContext('YOUTUBE_API_KEY');

  import { onMount, afterUpdate } from 'svelte';
import { prevent_default } from 'svelte/internal';
  let rootElm; //현재 컴포넌트의 루트 엘리먼트
  onMount(() => {
    console.log('the component has mounted');
    // IFrame Player API 코드 비동기 로드
    let tag = document.createElement('script');
    tag.src = "https://www.youtube.com/iframe_api";
    rootElm.appendChild(tag);
  });
  afterUpdate(() => Sortable.create(document.querySelector('.sortable'), {
    handle: '.handle', //드래그핸들러엘리먼트선택자
    // draggable: "드래그가능한엘리먼트선택자",
    // filter: "드래그불가능한엘리먼트선택자", 
    // onEnd: function (ev) { console.log('onEnd:', evt.oldIndex,evt.newIndex); },
    onUpdate: function (ev){ //목록내의 순서가 변경된 경우
      const temp = songList.splice(ev.oldIndex, 1)[0]; //이동할요소를배열에서빼내서
      songList.splice(ev.newIndex, 0, temp); //배열의새로운위치에추가
      if (songIdx===ev.oldIndex) songIdx = ev.newIndex; 
      else if (ev.oldIndex < songIdx && songIdx <= ev.newIndex) songIdx--;
      else if (ev.newIndex <= songIdx && songIdx < ev.oldIndex) songIdx++;
      songList = songList;
      console.log('onUpdate:', ev.oldIndex,ev.newIndex,songIdx); 
    }
  }));

  let songList = []; //재생목록
  let songIdx = -1; //현재 재생 중인 곡 목록
  let songPlaying = false; //현재 재생 중인지 여부
  let songBuffering = false; //현재 버퍼링 중인지 여부
  let songTimer; //1초마다 재생 진행을 출력하기 위한 타이머아이디
  let songUrl = 'https://youtu.be/DN20QSP3CqI'; //사용자에게입력받은동영상주소
  
  let searchWord;
  let searchList = []; //검색결과 리소스({ type: , id: , title: , img:  })들을 저장
  function searchSong(event) { //검색창에 키보드 입력시 실행
    // if (event.key !== 'Enter') return; //엔터키를 입력한 경우에만 검색 작업 수행
    // event.preventDefault(); //엔터키는 입력요소의 값에 반영하지 않음
    if ((!searchWord)||searchWord.trim().length===0) return; //검색어가 존재하는 경우에만 검색 작업 수행

    //검색어가 유튜브 URL 형태라면 즉시 추가하고, 그렇지 않으면 검색 작업 수행
    if (addUrl(searchWord)) return; //검색어가 주소라면 처리하고 종료
    
    //유튜브 데이터 검색을 위한 파라미터 설정 
    const params = new URLSearchParams();
    params.set('key', youtubeApiKey );
    params.set('part', 'id,snippet' );
    params.set('fields', 'items(id,snippet(title,thumbnails))' );
    params.set('type', 'video,playlist' ); //기본값은 video,channel,playlist
    params.set('maxResults', 20 );
    params.set('q', searchWord );
    //유튜브 데이터 검색
    fetch('https://www.googleapis.com/youtube/v3/search?'+params.toString(),{
      // method: 'GET', // *GET, POST, PUT, DELETE, etc.
      mode: 'cors', // no-cors, cors, *same-origin
      // cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
      // credentials: 'same-origin', // include, *same-origin, omit
    })
    .then(resp => resp.json()) //응답 내용을 JSON으로 파싱하는 resp.json() 역시 비동기 함수이므로 then() 으로 결과를 받아야 한다
    .then(data => {
      console.log(data);
      // { items : [  {  id: { kind: "youtube#video", videoId: "P2QOxJzvBZQ" }, snippet: { title: "", thumbnails: {} }  },  {  id: { kind: "youtube#playlist", playlistId: "PLzzbccz8B9wTWg5GE2aSZCX639LNIkZMS" }, snippet: { title: "", thumbnails: {} } }, ... ] }
      searchList = data.items.map(item=>{
        const kind = item.id.kind.split('#')[1]; //kind 정보에서 # 뒷부분 문자열 값을 타입으로 사용 ('video' 또는 'playlist')
        const id = item.id[kind+'Id']; //타입에 따라서 'videoId' 또는 'playlistId' 속성값을 아이디로 사용
        return { type: kind, id: id, title: item.snippet.title, img: item.snippet.thumbnails.default.url };
      }); //검색결과를 searchList 에 저장
    }).catch(err => {
      console.error(err);
      alert('정보를 조회할 수 없습니다 : ' + songUrl);
    });
  }

  function addToSongList(rsc) { 
    if (rsc.type==='playlist') { //추가할 리소스가 재생목록인 경우
      addPlaylistToSongList(rsc);
    }else {  //추가할 리소스가 동영상인 경우
      addVideoToSongList(rsc);
    }
  }

  function addPlaylistToSongList(rsc) {  //playlist 리소스를 SongList에 추가
    if (!confirm('재생목록에 있는 모든 곡들을 추가할까요?(최대50곡까지)')) return;
    //재생목록에 속한 동영상 정보 조회
    const params = new URLSearchParams();
    params.set('key', youtubeApiKey );
    params.set('part', 'snippet' ); //id,snippet,contentDetails,status
    params.set('fields', 'items(snippet(resourceId))' ); //동영상 길이 정보때문에 어차피 video 리소스 조회를 해야하므로 ID만 조회
    params.set('maxResults', 50); 
    params.set('playlistId', rsc.id); 
    fetch('https://www.googleapis.com/youtube/v3/playlistItems?'+params.toString(),{
      // method: 'GET', // *GET, POST, PUT, DELETE, etc.
      mode: 'cors', // no-cors, cors, *same-origin
      // cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
      // credentials: 'same-origin', // include, *same-origin, omit
    }).then(resp => resp.json()) //응답 내용을 JSON으로 파싱하는 resp.json() 역시 비동기 함수이므로 then() 으로 결과를 받아야 한다
    .then(data => {
      console.log(data);
      //재생목록에 포함된 리소스들 중 동영상 리소스에 대해서
      // const itemList = 
      data.items.filter(item=>item.snippet.resourceId.kind==='youtube#video')
                            .map(item => ({ id: item.snippet.resourceId.videoId })).forEach(item => addVideoToSongList(item));
                            // .map(item => ({ id: item.snippet.resourceId.videoId, title: item.snippet.title, img: item.snippet.thumbnails.default.url })) //, start: item.contentDetails.startAt, end: item.contentDetails.endAt
                            // .map(item => { item.duration = item.end - item.start; return item; });
      // songList = songList.concat( itemList );
    }).catch(err => {
      console.error(err);
      alert('정보를 조회할 수 없습니다 : ' + songUrl);
    });
  }

  function addVideoToSongList(rsc) {  //video 리소스를 SongList에 추가
    //동영상 상세 정보 조회
    const params = new URLSearchParams();
    params.set('key', youtubeApiKey );
    params.set('part', 'snippet,contentDetails' ); //id, snippet, contentDetails, fileDetails, liveStreamingDetails, player, processingDetails, recordingDetails, statistics, status, suggestions, topicDetails
    params.set('fields', 'items(id,snippet(title,thumbnails),contentDetails(duration))' ); //fileDetails(durationMs)는 동영상 소유자만 검색 가능
    params.set('id', rsc.id); 
    fetch('https://www.googleapis.com/youtube/v3/videos?'+params.toString(),{
      // method: 'GET', // *GET, POST, PUT, DELETE, etc.
      mode: 'cors', // no-cors, cors, *same-origin
      // cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
      // credentials: 'same-origin', // include, *same-origin, omit
    }).then(resp => resp.json()) //응답 내용을 JSON으로 파싱하는 resp.json() 역시 비동기 함수이므로 then() 으로 결과를 받아야 한다
    .then(data => {
      console.log(data);
      const item = data.items[0];
      const song = { id: item.id, title: item.snippet.title, duration: moment.duration(item.contentDetails.duration).asSeconds(), img: item.snippet.thumbnails.default.url };
      song.start = 0;
      song.end = song.duration;
      songList = songList.concat( song );
      songUrl = '';
    }).catch(err => {
      console.error(err);
      alert('정보를 조회할 수 없습니다 : ' + songUrl);
    });
  }

  function addUrl(url) {
    let rsc = { type: 'video' };
    switch (true) {
      case url.indexOf('youtu.be/') > 0 : //'https://www.youtube.com/playlist?list=재생목록ID'
        rsc.id = new URL(url).pathname.split('/').pop(); //마지막 경로 세그먼트가 동영상아이디
        break;
      case url.indexOf('youtube.com/watch') > 0 : //'https://www.youtube.com/watch?v=동영상ID'
        rsc.id = new URLSearchParams(new URL(url).search).get('v'); //v 파라미터 값이 동영상아이디
        break;
      case url.indexOf('youtube.com/playlist') > 0 : //'https://www.youtube.com/playlist?list=재생목록ID', 'https://www.youtube.com/playlist?list=재생목록ID'
        rsc.id = new URLSearchParams(new URL(url).search).get('list'); //list 파라미터 값이 재생목록아이디
        rsc.type = 'playlist';
        break;
      default:
        return false; //주소 형식이 아닌 경우 false 반환
    }
    addToSongList(rsc);
    searchWord = ''; //검색어
    return true; //주소 형식인 경우 true 반환
  }

  //API 코드 다운로드가 완료되면 <iframe> (유튜브 플레이어) 생성
  let player;
  window.onYouTubeIframeAPIReady = function onYouTubeIframeAPIReady() { // 전역함수로 등록 필요
    console.log('onYouTubeIframeAPIReady!');
    player = new YT.Player('player', {
      width: '320', //동영상 플레이어의 너비 (기본값은 640)
      height: '180', //동영상 플레이어의 높이 (기본값은 360)
      videoId: 'M7lc1UVf-VE', //플레이어에서 로드할 동영상
      playerVars: { // 플레이어 매개변수 설정
        enablejsapi: 1, // IFrame 또는 JavaScript Player API 호출을 통해 플레이어가 제어 가능 여부
        controls: 0, // 동영상 플레이어 컨트롤 표시 여부
        disablekb: 0, // 플레이어 키보드 컨트롤 가능 여부
        fs: 0, //전체 화면 버튼 표시 여부
        modestbranding: 1, //YouTube 로고를 컨트롤바에서 제거
        rel: 0, //초기 동영상 재생 종료시, 플레이어에 관련 동영상 표시 여부
        // start: 특정 시간(단위: 초) 이후에 동영상 재생을 시작 (지정한 시간에 가장 가까운 키프레임을 찾음, 일반적으로 2초 이내)
        // end: 동영상 재생을 중지해야 할 시간을 동영상 시작 부분부터 초 단위로 측정하여 지정
        iv_load_policy: 3 //동영상 특수효과 표시 여부
      },
      events: { //객체의 속성은 API가 실행하는 이벤트명과 이벤트 리스너(함수)
        // 'onReady': onPlayerReady,
        'onStateChange': onPlayerStateChange
      }
    });
  }
  
  // 비디오 플레이어가 준비되면 실행할 함수
  function startVideo(event) {
    console.log('startVideo!');
    if (songPlaying){ //현재 재생 중인 경우
      player.pauseVideo();
    }else{  //현재 재생 중이 아닌 경우
      if (!songList.length) { //재생목록이 비어있는 경우
        alert('재생할 음악이 없습니다.');
        return;
      }
      if (songIdx===-1) { //현재 재생목록에서 재생한 적이 없는 경우
        if (++songIdx >= songList.length) songIdx = 0; //재생 중에 종료된 경우, 다음곡 설정(없는 경우 첫곡으로 돌아가기)
        player.loadVideoById({
        videoId:songList[songIdx].id,
          // startSeconds:Number,
          // endSeconds:Number,
          // suggestedQuality:String 
        });
      }
      player.playVideo();  
    }
    songPlaying = !songPlaying;
  }

  function clickSong(idx) {
    songIdx = idx; 
    player.loadVideoById({
    videoId:songList[songIdx].id,
      // startSeconds:Number,
      // endSeconds:Number,
      // suggestedQuality:String 
    });
    player.playVideo();
    songPlaying = true;
  }

  function delSong(idx) {
    if (!confirm('정말 삭제할까요?')) return; //삭제 의지 재확인
    
    songList = songList.filter((v,i,a)=>i!==idx); //곡 삭제
    
    if (idx < songIdx) songIdx--; //현재 재생곡보다 앞쪽의 곡을 삭제한 경우 현재 재생곡 번호를 1 감소
    else if (idx === songIdx) { //현재 재생곡을 삭제한 경우
      if (songList.length===0) { //더 이상 곡이 없는 경우
        songIdx = -1; //재생곡 번호를 -1로 초기화
        if (songPlaying) { //재생중이었다면
          songPlaying = false; //재생 중단 상태 표시
          player.stopVideo(); //재생 중단
        }
      } else { //곡이 있는 경우
        if (songIdx === songList.length) songIdx = 0; //삭제한 재생곡이 마지막 곡이었던 경우 재생곡 번호를 첫 곡으로 설정
        //변경된 재생곡을 로드하고
        player.loadVideoById({ 
          videoId:songList[songIdx].id,
            // startSeconds:Number,
            // endSeconds:Number,
            // suggestedQuality:String 
          });
        if (songPlaying) player.playVideo(); //재생 중이 었다면 재생 시작
      } 
    }
  }

  // 비디오 플레이어의 상태가 변할 때 실행할 함수
  function onPlayerStateChange(event) {
    console.log('onPlayerStateChange!', event.target, event.data); //플레이어객체와 플레이어상태
    songBuffering = false;
    switch (event.data) { // === player.getPlayerState()
      case -1: //시작되지 않음 (플레이어가 동영상을 처음 로드)
        break;
      case 0: //종료 === YT.PlayerState.ENDED
        if (++songIdx >= songList.length) songIdx = 0; //재생 중에 종료된 경우, 다음곡 설정(없는 경우 첫곡으로 돌아가기)
        player.loadVideoById({
        videoId:songList[songIdx].id,
          // startSeconds:Number,
          // endSeconds:Number,
          // suggestedQuality:String 
        });
        player.playVideo();
        break;
      case 1: //재생 중 === YT.PlayerState.PLAYING
        seekMax = player.getDuration(); 
        songTimer = setInterval(function() {
            // console.log(player.getCurrentTime(), player.getDuration());
            seekVal = player.getCurrentTime();
        }, 1000);
        break;
      case 2: //일시중지 === YT.PlayerState.PAUSED
        clearInterval(songTimer);
        break;
      case 3: //버퍼링 === YT.PlayerState.BUFFERING
        songBuffering = true;
        break;
      case 5: //동영상 신호 (동영상이 신호를 받고 재생할 준비가 되면) === YT.PlayerState.CUED
        break;
    }
  }

  let seekMin = 0;
	let seekMax = 0;
	let seekVal = 0;
	function seekChange(){
    player.seekTo(seekVal, true);
	}

</script>

<style> 
  /* 재생목록의 seekbar 너비를 패널 전체 너비로 */
  input[type="range"] {
    width: 100%;
  }

  /* 재생목록의 디스크 아이콘의 마우스 포인터를 클릭 가능한 모양으로 */
  .panel .panel-icon {
    cursor: pointer;
  }

  /* panel-block 내부에서 columns 가 전체 영역을 차지하도록 설정 */
  .panel-block .columns {
    width: 100%;
    margin: 0px; /* .columns 의 디폴트 음수 마진 제거 */
  }

  /* 패널 아이콘의 디폴트 우측 마진을 제거 */
  .panel-icon {
    margin-right: 0px;
  }

  .sortable {
    max-height:100%;
    overflow-y:auto;
  }
</style>