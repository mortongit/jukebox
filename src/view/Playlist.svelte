<div bind:this="{rootElm}" class="columns">
  <div  class="column">
    <nav class="panel is-info">
      <div class="panel-heading">재생목록</div>
      {#each songList as item,idx}
        <!-- svelte-ignore a11y-missing-attribute -->
        <div class="panel-block {songIdx===idx?'is-active':''}" >
          <span on:click="{ev=>clickSong(idx)}" class="panel-icon  fa-2x">
            <i class="fas fa-compact-disc {songIdx===idx?'fa-spin':''}" aria-hidden="true"></i>
          </span>
          <!-- <i class="fas fa-record-vinyl"></i> -->
          <div>
            <p class="title is-6">{item.title}</p>
            <!-- <p >{item.duration} : {item.id} : {item.url}</p> -->
          </div>
          <button on:click="{ev=>delSong(idx)}" class="button">
            <i class="fas fa-times" aria-hidden="true"></i>
          </button>
        </div>
      {/each}
      <div class="panel-block">
        <input type="url" bind:value="{songUrl}" placeholder="VIDEO URL" class="input" />
        <button type="button" on:click="{addSong}" class="button is-info"><i class="fas fa-plus bi-plus"></i></button>
      </div>
      <div class="panel-block">
        <button type="button" on:click="{startVideo}" class="button is-info is-fullwidth"><i class="fas {songPlaying?'fa-pause':'fa-play'} {songPlaying?'bi-pause':'bi-play'}"></i></button>
      </div>
      <div class="panel-block">
        <input type="range" min="{seekMin}" max="{seekMax}" bind:value="{seekVal}" on:change={seekChange} class="form-range" >
      </div>
    </nav>
    <!-- <button on:click="{stopVideo}" class="button"><i class="fas fa-stop"></i></button> -->
    <!-- <iframe> (과 유튜브 플레이어)로 대체할 <div> 엘리먼트 -->
    <!-- <iframe id="player" bind:this="{playerElm}" width="100" height="100" src="https://www.youtube.com/embed/DN20QSP3CqI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe> -->
    
    <div id="player"></div>
  </div>
  <div  class="column">
    <nav class="panel is-info">
      <div class="panel-heading">검색</div>
      <div class="panel-block">
        <p class="control has-icons-left">
          <input class="input" type="text" placeholder="Search" >
          <span class="icon is-left">
            <i class="fas fa-search" aria-hidden="true"></i>
          </span>
        </p>
      </div>
    </nav>
  </div>
</div>

<script>
  // https://youtu.be/DN20QSP3CqI
  // https://youtu.be/Sl5I4I0ZDrY
  // https://youtu.be/lpwG8f9nt4s
  // https://www.youtube.com/watch?v=DN20QSP3CqI

  import { getContext } from 'svelte'
  const youtubeApiKey = getContext('YOUTUBE_API_KEY');


  import { onMount } from 'svelte';
  let rootElm; //현재 컴포넌트의 루트 엘리먼트
  onMount(() => {
    console.log('the component has mounted');
    // IFrame Player API 코드 비동기 로드
    let tag = document.createElement('script');
    tag.src = "https://www.youtube.com/iframe_api";
    rootElm.appendChild(tag);
  });

  let songList = []; //재생목록
  let songIdx = -1; //현재 재생 중인 곡 목록
  let songPlaying = false; //현재 재생 중인지 여부
  let songTimer; //1초마다 재생 진행을 출력하기 위한 타이머아이디
  let songUrl = ''; //사용자에게입력받은동영상주소
  
  function addToSongList(videoId) {
    let params = new URLSearchParams();
    params.set('url', 'https://www.youtube.com/watch?v='+videoId);
    params.set('format', 'json');
    
    fetch('https://www.youtube.com/oembed?'+params.toString(),{
      // method: 'GET', // *GET, POST, PUT, DELETE, etc.
      mode: 'cors', // no-cors, cors, *same-origin
      // cache: 'no-cache', // *default, no-cache, reload, force-cache, only-if-cached
      // credentials: 'same-origin', // include, *same-origin, omit
    }).then(resp => resp.json()) //응답 내용을 JSON으로 파싱하는 resp.json() 역시 비동기 함수이므로 then() 으로 결과를 받아야 한다
    .then(data => {
      console.log(data);
      songList = songList.concat( { url: songUrl, id: videoId, title: data.title, img: data.thumbnail_url } );
      songUrl = '';
    }).catch(err => {
      console.error(err);
      alert('정보를 조회할 수 없습니다 : ' + songUrl);
    });
  }

  function addSong(e) {
    let videoId;
    switch (true) {
      case songUrl.indexOf('://youtu.be/') > 0 :
        videoId = new URL(songUrl).pathname.split('/').pop();
        break;
      case songUrl.indexOf('://www.youtube.com/watch') > 0 :
        videoId = new URLSearchParams(new URL(songUrl).search).get('v');
        break;
      default:
        alert('올바른 주소 형식이 아닙니다 : ' + songUrl);
        return;
    }

    addToSongList(videoId);
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
		// alert(val);
	}

  // function stopVideo() {
  //   player.stopVideo();
  //   // player.clearVideo();
  // }

</script>

<style>
  input[type="range"] {
    width: 100%;
  }
</style>