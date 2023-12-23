<template>
  <h1 class="mb-1">
    {{ product ? product.product_name : '' }}
    <v-progress-circular v-if="loading" indeterminate :size="30"></v-progress-circular>
  </h1>

  <v-row>
    <v-col cols="12" sm="6">
      <PriceCard v-if="product" :product="product" elevation="1"></PriceCard>
    </v-col>
  </v-row>

  <br />

  <h2 class="mb-1">
    Product details
  </h2>

  <v-row>
    <v-col cols="12" sm="6">
      <v-btn v-if="product" size="small" append-icon="mdi-open-in-new" href="https://world.openfoodfacts.org/product/{{ product.code }}" target="_blank">
        Open Food Facts
      </v-btn>
    </v-col>
  </v-row>

  <br />

  <h2 class="mb-1">
    Last prices
    <small>{{ productPriceCount }}</small>
  </h2>

  <v-row>
    <v-col cols="12" sm="6" md="4" v-for="price in productPriceList" :key="price">
      <PriceCard :price="price" elevation="1" height="100%"></PriceCard>
    </v-col>
  </v-row>
</template>

<script>
import api from '../services/api'
import PriceCard from '../components/PriceCard.vue'

export default {
  components: {
    PriceCard,
  },
  data() {
    return {
      product: null,
      productPriceList: [],
      productPriceCount: null,
      loading: false,
    }
  },
  mounted() {
    this.getProduct(),
    this.getProductPrices()
  },
  methods: {
    getProduct() {
      this.loading = true
      return api.getProductById(this.$route.params.id)
        .then((data) => {
          this.product = data
          this.loading = false
        })
    },
    getProductPrices() {
      this.loading = true
      return api.getPrices({ product_id: this.$route.params.id, order_by: '-created' })
        .then((data) => {
          this.productPriceList = data.items
          this.productPriceCount = data.total
          this.loading = false
        })
    }
  }
}
</script>