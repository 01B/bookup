<template>
  <v-infinite-scroll :loading="loading" @bottom="nextPage"
                     style="max-height: 80vh; overflow-y: scroll;">

    <div class="container" v-for="book in content">
      <div class="row" v-on:click="clickBook(book)">
        <div class="col-md-2">
          <a href="#">
            <img class="img-book img-fluid rounded mb-3 mb-md-0" v-bind:src="book.image">
          </a>
        </div>
        <div class="col-md-10">
          <h3 class="book-title">{{ book.title }}</h3>
          <p class="book-description">{{ book.description }}</p>
          <p class="book-description">가격 : {{ book.price }} 원</p>
        </div>
      </div>
      <hr>
    </div>
  </v-infinite-scroll>
</template>

<script>
  import Vue from 'vue'
  import InfiniteScroll from 'v-infinite-scroll'
  import 'v-infinite-scroll/dist/v-infinite-scroll.css'
  import api from '@/api'
  import StoreModel from '../../models/StoreModel'

  Vue.use(InfiniteScroll)

  export default {
    name: 'search',

    data () {
      return {
        loading: false,
        keyword: '',
        pageable: {
          page: 0,
          last: false
        },
        content: []
      }
    },

    created () {
      this.$eventBus.$on('bookSearchResponse', this.onReceiveResponse)
      this.$eventBus.$on('keyword', this.onReceiveKeyword)
    },

    methods: {
      nextPage () {
        if (!this.pageable.last) {
          this.loading = true
          this.pageable.page++
          this.request()
          this.loading = false
        }
      },

      async request () {
        this.$eventBus.$on('keyword', this.onReceiveKeyword)
        let response = await api.get('books/' + this.keyword + '?page=' + this.pageable.page + '&size=20')
        this.content.push.apply(this.content, response.content)
        this.pageable.last = response.last
      },

      onReceiveResponse (response) {
        this.content = response.content
        this.pageable.page++
        this.pageable.last = response.last
      },

      onReceiveKeyword (keyword) {
        this.keyword = keyword
      },

      clickBook (book) {
        StoreModel.getStores(book.isbn).then(stores => {
          if (stores.length <= 0) {
            alert('모든 서점에서 도서를 찾을 수 없습니다.')
            return
          }

          let message = ''

          stores.forEach(function (store) {
            message += store.storeName + '\n'
          })

          alert(message)
        })
      }
    }
  }
</script>

<style>
  .book-title {
    text-align: left;
  }

  .book-description {
    text-align: left;
  }

  .img-book {
    width: 118px;
    height: 160px;
    margin: 0 auto;
  }
</style>