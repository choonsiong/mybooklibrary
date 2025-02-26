<template>
  <div class="p-4 bg-green-100">
    <div class="w-[80%] mx-auto pt-10">
      <p class="font-bold text-center text-3xl">{{ book.title }}</p>
      <div class="m-2.5 overflow-hidden rounded-md h-80 flex justify-center items-center">
        <img class="w-full h-full object-scale-down" :src="appEnvironment.imageURL() + '/' + book.slug + '.jpg'"
             alt="profile-picture" />
      </div>
      <p class="mt-3 text-center font-light text-xl">{{ allAuthors }}</p>
      <p class="mt-5 text-center font-bold text-2xl">{{ normalizedPublisherName }} ({{ book.publication_year }}) <br><span class="font-light text-xl text-gray-500">ISBN {{ book.isbn }}</span></p>
      <p class="mt-10 font-extralight">{{ book.description }}</p>
      <div class="mt-10">
        <div class="flex flex-wrap mx-3 pb-3 pt-2 px-1">
          <span v-for="genre in book.genres"
                class="text-sm text-slate-600 font-medium me-2 mb-2 border-2 rounded-md p-1">{{ genre.genre_name.toLowerCase()
            }}</span>
        </div>
      </div>
      <div v-if="reviews !== null && reviews.length > 0">
        <div v-for="review in reviews" :key="review.id" class="p-5 bg-green-50 mb-3 rounded-2xl">
          <p class="mb-1 font-light">{{ review.created_at.slice(0, 19).replace('T', ' ') }} by {{ review.user_name }}</p>
          <em>{{ review.review }}</em>
        </div>
      </div>
      <div class="text-center">
        <button v-if="store.isLoggedIn && showWriteReview" class="bg-green-400 p-3 rounded-xl font-bold text-white" @click="handleWriteReview">Write a Review</button>
      </div>
    </div>
  </div>
</template>
<script>
import { store } from '@/store.js'
import appEnvironment from '@/environment.js'
import router from '@/router/index.js'
import Security from '@/security.js'

export default {
  name: 'BookDetail',
  data() {
    return {
      book: store.book,
      showWriteReview: true,
      reviews: {}
    }
  },
  computed: {
    store() {
      return store
    },
    appEnvironment() {
      return appEnvironment
    },
    allAuthors() {
      let result = ''
      if (store.book.authors.length == 1) {
        return this.capitalizedEachWord(store.book.authors[0].author_name)
      } else {
        for (let i = 0; i < store.book.authors.length; i++) {
          result += store.book.authors[i].author_name
          if (i !== store.book.authors.length - 1) {
            result += ', '
          }
        }
        return this.capitalizedEachWord(result)
      }
    },
    normalizedPublisherName() {
      return this.capitalizedEachWord(store.book.publisher.publisher_name)
    }
  },
  methods: {
    capitalizedEachWord(words) {
      const arr = words.split(' ')
      for (let i = 0; i < arr.length; i++) {
        arr[i] = arr[i][0].toUpperCase() + arr[i].substring(1)
      }
      return arr.join(' ')
    },
    handleWriteReview() {
      store.book = this.book
      router.push('/manage/reviews/new')
    },
  },
  beforeMount() {
    fetch(appEnvironment.apiURL() + '/reviews/' + store.book.id)
      .then((resp) => resp.json())
      .then((jsonResp) => {
        if (jsonResp.error) {
          notie.alert({
            type: 'error',
            text: jsonResp.message
          })
        } else {
          this.reviews = jsonResp.data.results
        }
      })
      .catch((err) => {
        notie.alert({
          type: 'error',
          text: err
        })
      })

    if (store.isLoggedIn) {
      const payload = {
        user_id: store.user.id,
        book_id: store.book.id,
      }

      fetch(appEnvironment.apiURL() + '/admin/reviews/exists', Security.requestOptions(payload))
        .then((resp) => resp.json())
        .then((jsonResp) => {
          if (jsonResp.error) {
            notie.alert({
              type: 'error',
              text: jsonResp.message
            })
          } else {
            this.showWriteReview = !jsonResp.data.exists;
          }
        })
        .catch((err) => {
          notie.alert({
            type: 'error',
            text: err
          })
        })
    }
  }
}
</script>