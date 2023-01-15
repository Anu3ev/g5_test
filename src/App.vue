<template>
  <div id="app" v-cloak>
    <div ref="swiper" class="swiper">
      <div class="swiper-wrapper">
        <div
          v-for="image, i in images"
          :key="i" 
          class="swiper-slide"
        >
          <img :src="image">
        </div>
      </div>
      <div class="swiper-pagination"></div>
  
      <div class="swiper-button-prev"></div>
      <div class="swiper-button-next"></div>
    </div>

    <div class="products">
      <app-product-card 
        v-for="product, i in generatedProducts"
        :key="product.id"
        :iteration="++i"
        :product="product"
      />
    </div>
  </div>
</template>

<script>
const IMAGES_REQUEST_URL = 'https://random.dog/woof.json'
const IMAGES_COOKIE_NAME = 'slides'
const PRODUCTS_REQUEST_URL = 'https://dummyjson.com/products?limit=20'
const PRODUCTS_STORAGE_NAME = 'products'

// Import Swiper and plugins
import Swiper, { Navigation, Pagination } from 'swiper'

// Import Swiper styles
import 'swiper/css';
import 'swiper/css/navigation';
import 'swiper/css/pagination';
import AppProductCard from './components/app-product-card.vue'

export default {
  name: "App",
  data: () => ({
    images: [],
    products: null
  }),
  components: {
    AppProductCard
  },
  created() {
    if(!localStorage[PRODUCTS_STORAGE_NAME]) {
      this.getProducts()
    } else {
      this.products = localStorage[PRODUCTS_STORAGE_NAME]
    }
  },
  mounted() {
    // Проверяем что картинки для слайдера есть в кукисах и если нет, то вызываем метод для их получения
    if (this.$cookies.isKey(IMAGES_COOKIE_NAME)) {
      this.images = this.$cookies.get(IMAGES_COOKIE_NAME).split(',')
      this.initSwiper()
    } else {
      this.getImages()
    }
  },
  computed: {
    generatedProducts() {
      return JSON.parse(this.products)
    }
  },
  methods: {
    getImages() {
      fetch(IMAGES_REQUEST_URL)
        .then(response => response.json())
        .then(data => {
          // Когда получено нужное кол-во изображений вызываем метод для их сохранения в кукисах
          if (this.images.length === 10) {
            this.saveImagesCookies()
            this.initSwiper()
            return
          }

          // Проверяем что нам не прилетело видео
          if (data.url.match(/\.(mp4|webm)$/)) {
            this.getImages()
            return
          }

          // Пушим URL каждой картинки в массив images, и повторно вызываем метод
          this.images.push(data.url)
          this.getImages()
        });
    },
    getProducts() {
      // Получаем товары и кладём их в localStorage, а затем в products
      fetch(PRODUCTS_REQUEST_URL)
      .then(response => response.json())
      .then(data => {
        localStorage.setItem(PRODUCTS_STORAGE_NAME, JSON.stringify(data.products))
        this.products = localStorage.getItem(PRODUCTS_STORAGE_NAME)
      })
    },
    saveImagesCookies() {
      this.$cookies.set(IMAGES_COOKIE_NAME, this.images, 300)
    },
    initSwiper() {
      new Swiper(this.$refs.swiper, {
        modules: [Navigation, Pagination],
        loop: false,
        slidesPerView: 1,
        spaceBetween: 30,
        pagination: {
          el: ".swiper-pagination",
          clickable: true,
        },

        navigation: {
          nextEl: '.swiper-button-next',
          prevEl: '.swiper-button-prev',
        },

        scrollbar: {
          el: '.swiper-scrollbar',
        },
        breakpoints: {
          768: {
            slidesPerView: 2
          },
          1440: {
            slidesPerView: 3
          }
        },
      })
    }
  }
};
</script>

<style lang="scss">
.swiper-pagination-bullet:not(.swiper-pagination-bullet-active) {
  background-color: #fff;
  opacity: 1;
}

img {
  width: 600px;
  height: 500px;
  object-fit: cover;
}

.products {
  display: grid;
  grid-template-columns: repeat(auto-fill, 300px) 300px;
  grid-gap: 15px;
  margin-top: 2em;
}
</style>
