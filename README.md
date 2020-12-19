
# Vue-Node_Study

### Node.js와 Vue.js를 연계하여 서비스 제작
:boy: Author: 유대성[(daerong)](https://github.com/daerong/)  

## 1. 개요  
* Express
* Vue.js

## Vue-Express 통합 프로젝트 만들기

### 1. express 설치
```
npm install express-generator -g
express --view=jade backend
cd backend
npm install
npm start
```
완료 후 화면
![그림1](https://user-images.githubusercontent.com/26676087/102683522-2b1d0280-4215-11eb-9bb4-87d3874ab9bc.PNG)

### 2. vue-cli 설치
```
npm install -g vue-cli
vue init webpack frontend
cd frontend
npm install
npm run dev
```
vue init webpack frontend 설정
![그림2](https://user-images.githubusercontent.com/26676087/102683526-3a03b500-4215-11eb-8117-3da5c0f05d4e.PNG)
완료 후 화면
![그림3](https://user-images.githubusercontent.com/26676087/102683534-45ef7700-4215-11eb-9616-fb582fc8de23.PNG)

### 3. json data 추가
backend/data/movies.json를 생성하여 다음을 작성.
```
[
    {
        "id": 1,
        "name": "공조",
        "year": 2017,
        "director": "김성훈",
        "poster": "http://img.cgv.co.kr/Movie/Thumbnail/Poster/000079/79416/79416_185.jpg",
        "description" : "공조 입니다."
    },
    {
        "id": 2,
        "name": "컨택트",
        "year": 2017,
        "director": "드니 빌뇌브",
        "poster": "http://img.cgv.co.kr/Movie/Thumbnail/Poster/000079/79437/79437_185.jpg",
        "description" : "컨택트 입니다."
    },
    {
        "id": 3,
        "name": "더킹",
        "year": 2017,
        "director": "한재림",
        "poster": "http://img.cgv.co.kr/Movie/Thumbnail/Poster/000079/79423/79423_185.jpg",
        "description" : "더킹 입니다."
    },
    {
        "id": 4,
        "name": "모아나",
        "year": 2017,
        "director": "론 클레멘츠, 존 머스커",
        "poster": "http://img.cgv.co.kr/Movie/Thumbnail/Poster/000079/79316/79316_185.jpg",
        "description" : "모아나 입니다."
    },
    {
        "id": 5,
        "name": "라이언",
        "year": 2017,
        "director": "가스 데이비스",
        "poster": "http://img.cgv.co.kr/Movie/Thumbnail/Poster/000079/79396/79396_185.jpg",
        "description" : "라이언 입니다."
    },
    {
        "id": 6,
        "name": "너의 이름은",
        "year": 2017,
        "director": "신카이 마코토",
        "poster": "http://img.cgv.co.kr/Movie/Thumbnail/Poster/000079/79313/79313_1000.jpg",
        "description" : "너의 이름은 입니다."
    }
]
```

### 4. backend route 설정
backend/routes/movie.js를 생성하여 다음을 작성.
```
const express = require('express');
const router = express.Router();
const movies = require('../data/movie.json');

router.get('/', function(req, res){
    res.send(movies);
});

router.get('/:id', function(req, res){
    const id = parseInt(req.params.id, 10);
    const movie = movies.filter(function(movie){
        return movie.id === id;
    });
    res.send(movie);
});

module.exports = router;
```
app.js에 다음 추가
```
var movieRouter = require('./routes/movie');
app.use('/movies', movieRouter);
```

### 5. Frontend implement
frontend/config/index.js에서 proxyTable를 찾아 다음과 같이 변경
```
proxyTable: {
    '/movies': {
        target: 'http://localhost:3000/movies',
        changeOrigin: true,
        pathRewrite: {
            '^/movies': ''
        }
    }
},
```

### 6. axios
frontend/src/main.js에 다음을 추가
```
import axios from 'axios'
Vue.prototype.$http = axios
```

### 7. Frontend page 추가
frontend/src/components/MovieListPage.vue를 추가하여 다음을 작성
```
<template>
  <div class="movies">
    <h1>영화 목록</h1>
    <div class="container">
      <div class="outer">
        <div class="inner">
          <div class="centered" v-for="movie in movies" :key="movie.id"> <img v-bind:src="movie.poster" class="poster">
            <div> <strong>{{movie.name}}</strong> - <i>{{movie.director}}</i> [{{movie.year}}] <router-link :to="{ name: 'detailmovie', params: { id: movie.id }}">더보기</router-link>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
/* eslint-disable */
export default { created() { this.$http.get("/movies").then(response => { this.movies = response.data; }); }, data() { return { movies: [] }; } };

</script>
<style>
.outer {
  display: table;
  width: 100%;
  height: 100%;
}

.inner {
  display: table-cell;
  vertical-align: middle;
  text-align: center;
}

.centered {
  position: relative;
  display: inline-block;
  width: 50%;
  padding: 1em;
  font-size: 1.5em;
  /* background: orange; */
  /* color: white; */
}

.poster {
  width: 30%;
  height: 40%;
}

</style>
```

frontend/src/components/DetailMoviePage.vue를 추가하여 다음을 작성
```
<template>
  <div>
    <h1>상세 내용</h1>
    <div class="container">
      <div class="outer">
        <div class="inner">
          <div class="centered"> <img v-bind:src="movie.poster" class="poster">
            <div> <strong>{{movie.name}}</strong> - <i>{{movie.director}}</i> [{{movie.year}}] <p>{{movie.description}}</p>
              <router-link :to="{ name: 'movielist' }">돌아가기</router-link>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
/* eslint-disable */
export default { created: function() { var id = this.$route.params.id;
    this.$http.get(`/movies/${id}`).then(response => { this.movie = response.data[0]; }); }, data: function() { return { movie: {} }; } };

</script>
```

### 8. Frontend index 수정
frontend/src/router/index.js를 다음과 같이 수정
```
/* eslint-disable */
import Vue from 'vue'
import Router from 'vue-router'
import MovieListPage from '@/components/MovieListPage'
import DetailMoviePage from '@/components/DetailMoviePage'

Vue.use(Router)

export default new Router({
  routes: [{
      path: '/',
      name: 'movielist',
      component: MovieListPage
    },
    {
      path: '/:id',
      name: 'detailmovie',
      component: DetailMoviePage
    }
  ]
})
```