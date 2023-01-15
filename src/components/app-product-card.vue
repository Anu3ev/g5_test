<template>
<div
  class="product-card"
  :class="isDesktop"
>
  <div 
    class="product-card__title"
    v-text="product.title"
  />
  <div
    class="product-card__brand"
    v-text="product.brand"
  />
  <div
    class="product-card__category"
    v-text="product.category" 
  />
  <img 
    class="product-card__img" 
    :src="product.thumbnail"
  />
  <div
    class="product-card__description"
    v-text="cuttedDescription" 
  />
  <div class="product-card__stock">
    Stock: {{ currentStock }}
  </div>

  <div
    v-if="generatedLabel"
    class="product-card__label"
    :class="`product-card__label--${generatedLabel}`"
    v-text="generatedLabel"
  >

  </div>
</div>
</template>

<script>
// Import Device Detector
import DeviceDetector from "device-detector-js";

const deviceDetector = new DeviceDetector();
const userAgent = window.navigator.userAgent;
const device = deviceDetector.parse(userAgent);

export default {
  name: 'AppProductCard',
  data: () => ({
    currentStock: ""
  }),
  props: {
    product: Object,
    iteration: Number
  },
  computed: {
    isDesktop() {
      return {
        'product-card--desktop': device.device.type === 'desktop'
      }
    },
    cuttedDescription() {
      return this.product.description.slice(0, 30) + '...'
    },
    generatedLabel() {
      switch(this.iteration) {
        case 1: return 'top'
        case 2: return 'sale'
        case 3: return 'popular'
        default: return ''
      }
    }
  },
  methods:{
    getCurrentStock: function (product) {
      let self = this;
      self.currentStock = product.stock

      let stocksChangeInterval = setInterval(() => {
        product.stock--
        self.currentStock = product.stock

        if(self.currentStock < Math.floor((Math.random() * 10) + 1)) {
        clearInterval(stocksChangeInterval)
      }
    }, Math.floor((Math.random() * 10) + 1) * 1000)
   }
  },
  mounted () {
    this.getCurrentStock(this.product)
  },
  beforeUnmount() {
    const interval_id = window.setInterval(function(){}, Number.MAX_SAFE_INTEGER);

    // Clear any timeout/interval up to that id
    for (let i = 1; i < interval_id; i++) {
      window.clearInterval(i);
    }
  }
}
</script>

<style lang="scss">
.product-card {
  display: grid;
  height: 300px;
  overflow: hidden;
  position: relative;
  border: 1px solid #c3c3c3;
  padding: 10px;
  border-radius: 5px;

  &--desktop {
    height: 350px;
  }

  &__title {
    font-weight: 600;
  }

  &__description {
    opacity: .5;
  }

  &__img {
    width: 100%;
    height: 200px;
  }

  &__label {
    position: absolute;
    top: 5px;
    right: 5px;
    color: white;
    padding: 5px 10px;
    border-radius: 5px;

    &--top {
      background: rgb(95, 95, 219);
    }
    &--sale {
      background: rgb(192, 28, 28);
    }
    &--popular {
      background: rgb(76, 199, 27);
    }
  }
}
</style>
