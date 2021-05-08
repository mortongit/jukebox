# MINI JUKEBOX APP

간단한 유튜브 음원 쥬크박스 앱 프로젝트

## Features
- 유튜브 동영상 및 재생목록 주소를 이용한 음원 추가
- 유튜브 동영상 및 재생목록 검색을 이용한 음원 추가
- 음원 삭제
- 음원 재생 순서 변경 (드래그앤드롭)
- 음원 재생 및 일시 정지
- 음원 목록 순차 재생, 목록 반복 재생, 선택 음원 재생
- 탐색바(seekbar)를 이용한 재생 위치 변경
- 재생목록 전체 곡수 및 시간 출력
- 음원 재생 구간 지정

## Tools and Libraries
- nodejs 14.16.0
- rollup 2.3.4
- svelte 3.0.0
- bulma@0.9.2
- bulma-switch@2.0.0
- font-awesome/5.15.3
- youtube player_api
- youtube data_api
- moment@2.29.1
- sortablejs@1.13.0
- svelte-nouislider@1.0.1
- firebase/8.4.1

## Get started

* 프로젝트 다운로드(클론) 후 의존성 설치 및 실행
```bash
$ npx degit mortongit/jukebox myapp
$ cd myapp
$ npm install
```

* 프로젝트 루트에 key.js 파일을 생성하고 유튜브 API 키를 설정
```javascript
export default {
    YOUTUBE_API_KEY: 'YOUTUBE API KEY'
}
```

* 프로젝트 실행 후 웹 브라우저에서 [localhost:5000](http://localhost:5000) 접속
```bash
$ npm run dev
```

## Building and running in production mode


```bash
npm run build
```


