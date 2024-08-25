# Vue.js

# 글 작성화면 만들기

.vscode 삭제

aboutview 삭제

components 안에 파일 삭제

npm i axios

npm i bootstrap@5.3.3

npm install element-plus --save

App.vue

```jsx
<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <header>
      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
      </nav>
  </header>

  <RouterView />
</template>

<style scoped>
</style>

```

index.ts

```jsx
const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: HomeView
    },
    {
      path : "/write",
      name : "write",
      component : WriteView
    }
    // {
    //   path: '/about',
    //   name: 'about',
    //   // route level code-splitting
    //   // this generates a separate chunk (About.[hash].js) for this route
    //   // which is lazy-loaded when the route is visited.
    //   component: () => import('../views/AboutView.vue')
    // }
  ]
})

export default router

```

WriteView.vue

```jsx
<script setup lang="ts">
import {ref} from "vue";
import axios from "axios";

const title = ref("")
const content = ref("")

const write = function (){
  axios.post("http://localhost:8080/posts",{
    title : title.value,
    content : content.value
  })
}

</script>

<template>

  <div>
    <el-input v-model="title" placeholder="제목을 입력해주세요"/>
  </div>

  <div class="mt-2">
    <el-input v-model="content" type="textarea" rows="15"></el-input>
  </div>

  <div class ="mt-2">
    <el-button type="primary" @click="write">글 작성완료</el-button>
  </div>

</template>

<style scoped>

</style>
```

# CORS 문제해결

CORS(Cross-Origin Resource Sharing) 오류는 브라우저에서 다른 도메인, 프로토콜, 또는 포트에서 리소스를 요청할 때 발생하는 보안 정책 위반으로 인해 발생합니다. 예를 들어, `http://example.com`에서 `http://api.example.com`으로 API 요청을 보낼 때, 두 도메인이 다르기 때문에 브라우저는 보안을 위해 이러한 요청을 기본적으로 차단합니다.

서버쪽에서 해결

```jsx
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("http://localhost:5173");
    }
}

```

프론트에서 해결

vite.config.ts

```jsx
import { fileURLToPath, URL } from 'node:url'

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import vueJsx from '@vitejs/plugin-vue-jsx'
import vueDevTools from 'vite-plugin-vue-devtools'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    vueJsx(),
    vueDevTools(),
  ],
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  },
  server: {
    proxy: {
      "/api":{
        target : "http://localhost:8080",
        rewrite: (path) => path.replace(/^\/api/, ''),
      },
    },
  },
});

```

# 글 리스트 화면 만들기

HomeVIew.vue

```jsx
<script setup lang="ts">
import axios from "axios";
import {ref} from "vue";

const posts= ref([]);

axios.get("api/posts?page=1&size=5").then((response)=>{

  response.data.forEach((r : any) =>{
    posts.value.push(r);
  })
});
</script>

<template>
  <main>
    <ul>
      <li v-for="post in posts" :key="post.id">

        <div>
          {{post.title}}
        </div>

        <div>
          {{post.content}}
        </div>
      </li>
    </ul>
  </main>
</template>

<style scoped>

  li{
    margin-bottom: 1rem;
  }

  li :last-child{
    margin-bottom: 0;
  }
</style>
```

# 글 내용 화면 만들기

index.ts

```jsx
import { createRouter, createWebHistory } from 'vue-router'
import HomeView from '../views/HomeView.vue'
import WriteView from '@/views/WriteView.vue'
import ReadView from "@/views/ReadView.vue";

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: HomeView
    },
    {
      path : "/write",
      name : "write",
      component : WriteView
    },
    {
      path : "/read/:postId",
      name : "read",
      component : ReadView,
      props: true
    }
    // {
    //   path: '/about',
    //   name: 'about',
    //   // route level code-splitting
    //   // this generates a separate chunk (About.[hash].js) for this route
    //   // which is lazy-loaded when the route is visited.
    //   component: () => import('../views/AboutView.vue')
    // }
  ]
})

export default router

```

ReadView.vue

```jsx
<script setup lang="ts">
import {defineProps, onMounted, ref} from "vue";
import axios from "axios";

const props = defineProps({
  postId: {
    type : [Number, String],
    require : true,
  }
})

const post = ref({
  id : 0,
  title : "",
  content : "",
});

onMounted(()=>{

  axios.get(`/api/posts/${props.postId}`).then((response)=>{

    post.value = response.data;

  });
})

</script>

<template>
  <h2>{{ post.title }}</h2>
  <h2>{{post.content}}</h2>
</template>

<style scoped>

</style>
```

# 글 수정 화면 만들기

index.ts

```jsx
import { createRouter, createWebHistory } from 'vue-router'
import HomeView from '../views/HomeView.vue'
import WriteView from '@/views/WriteView.vue'
import ReadView from "@/views/ReadView.vue";
import EditView from "@/views/EditView.vue";

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes: [
    {
      path: '/',
      name: 'home',
      component: HomeView
    },
    {
      path : "/write",
      name : "write",
      component : WriteView
    },
    {
      path : "/read/:postId",
      name : "read",
      component : ReadView,
      props: true
    },
    {
      path : "/edit/:postId",
      name : "edit",
      component : EditView,
      props: true
    }
    // {
    //   path: '/about',
    //   name: 'about',
    //   // route level code-splitting
    //   // this generates a separate chunk (About.[hash].js) for this route
    //   // which is lazy-loaded when the route is visited.
    //   component: () => import('../views/AboutView.vue')
    // }
  ]
})

export default router

```

ReadView.vue

```jsx
<script setup lang="ts">
import {defineProps, onMounted, ref} from "vue";
import axios from "axios";
import {useRouter} from "vue-router";

const props = defineProps({
  postId: {
    type : [Number, String],
    require : true,
  }
})

const post = ref({
  id : 0,
  title : "",
  content : "",
});

const router = useRouter();

const moveToEdit = () =>{
  router.push({name : "edit",params : {postId : props.postId}});
}

onMounted(()=>{

  axios.get(`/api/posts/${props.postId}`).then((response)=>{

    post.value = response.data;

  });
})

</script>

<template>
  <h2>{{ post.title }}</h2>
  <h2>{{post.content}}</h2>

  <el-button type="warning" @click="moveToEdit()">수정</el-button>
</template>

<style scoped>

</style>
```

EditView.vue

```jsx
<script setup lang="ts">
import {defineProps, ref} from "vue";
import axios from "axios";
import router from "@/router";

const post = ref({
  id : 0,
  title : "",
  content : "",
});

const props = defineProps({
  postId: {
    type : [Number, String],
    require : true,
  }
})

axios.get(`/api/posts/${props.postId}`).then((response)=>{

  post.value = response.data;

});

const edit = () =>{
  axios.patch(`/api/posts/${props.postId}`, post.value).then(()=>{

    router.replace({name: "home"})

  });
}

</script>

<template>

  <div>
    <el-input v-model="post.title" placeholder="제목을 입력해주세요"/>
  </div>

  <div class="mt-2">
    <el-input v-model="post.content" type="textarea" rows="15"></el-input>
  </div>

  <div class ="mt-2">
    <el-button type="warning" @click="edit()">글 수정완료</el-button>
  </div>

</template>

<style scoped>

</style>
```

# 화면 예쁘게 만들기