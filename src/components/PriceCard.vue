<template>
  <v-card :id="'price_' + price.id">
    <v-container class="pa-2">
      <v-row>
        <v-col v-if="!hideProductImage" style="max-width:15%">
          <v-img v-if="product && product.image_url" :src="product.image_url" style="max-height:50px;width:50px" @click="goToProduct()"></v-img>
          <v-img v-else :src="productImageDefault" style="height:50px;width:50px;filter:invert(.9);"></v-img>
        </v-col>
        <v-col :style="hideProductImage ? '' : 'max-width:85%'">
          <h3 v-if="!hideProductTitle" @click="goToProduct()">{{ getPriceProductTitle() }}</h3>

          <p v-if="!hideProductDetails && !hasCategoryTag" class="mb-2">
            <span v-if="hasProductCode">
              <PriceCountChip :count="product.price_count" @click="goToProduct()"></PriceCountChip>
              <span v-if="hasProductSource">
                <ProductBrands :productBrands="product.brands" :readonly="readonly"></ProductBrands>
                <ProductQuantityChip class="mr-1" :productQuantity="product.product_quantity" :productQuantityUnit="product.product_quantity_unit"></ProductQuantityChip>
              </span>
              <ProductMissingChip v-else></ProductMissingChip>
            </span>
            <span v-else>
              <PriceOrigins v-if="hasPriceOrigin" class="mr-1" :priceOrigins="price.origins_tags"></PriceOrigins>
              <PriceLabels v-if="hasPriceLabels" class="mr-1" :priceLabels="price.labels_tags"></PriceLabels>
            </span>
          </p>
        </v-col>
      </v-row>

      <PricePriceRow v-if="price" :price="price" :productQuantity="product ? product.product_quantity : null" :productQuantityUnit="product ? product.product_quantity_unit : null" :hidePriceDate="hidePriceDate"></PricePriceRow>

      <PriceFooterRow v-if="price && !hidePriceFooterRow" :price="price" :hidePriceLocation="hidePriceLocation" :hidePriceProof="hidePriceProof" :readonly="readonly"></PriceFooterRow>
    </v-container>
  </v-card>
</template>

<script>
import { defineAsyncComponent } from 'vue'
import utils from '../utils.js'

export default {
  components: {
    'PriceCountChip': defineAsyncComponent(() => import('../components/PriceCountChip.vue')),
    'ProductBrands': defineAsyncComponent(() => import('../components/ProductBrands.vue')),
    'ProductQuantityChip': defineAsyncComponent(() => import('../components/ProductQuantityChip.vue')),
    'ProductMissingChip': defineAsyncComponent(() => import('../components/ProductMissingChip.vue')),
    'PriceOrigins': defineAsyncComponent(() => import('../components/PriceOrigins.vue')),
    'PriceLabels': defineAsyncComponent(() => import('../components/PriceLabels.vue')),
    'PricePriceRow': defineAsyncComponent(() => import('../components/PricePriceRow.vue')),
    'PriceFooterRow': defineAsyncComponent(() => import('../components/PriceFooterRow.vue'))
  },
  props: {
    'price': null,
    'product': null,
    'hideProductImage': false,
    'hideProductTitle': false,
    'hideProductDetails': false,
    'hidePriceDate': false,
    'hidePriceFooterRow': false,
    'hidePriceLocation': false,
    'hidePriceProof': false,
    'readonly': false
  },
  data() {
    return {
      productImageDefault: 'https://world.openfoodfacts.org/images/icons/dist/packaging.svg',
    }
  },
  mounted() {
  },
  computed: {
    hasProduct() {
      return !!this.product
    },
    hasPrice() {
      return !!this.price
    },
    hasCategoryTag() {
      return !!this.price.category_tag
    },
    hasProductName() {
      return this.hasProduct && !!this.product.product_name
    },
    hasProductCode() {
      return this.hasProduct && !!this.product.code
    },
    hasProductSource() {
      return this.hasProduct && !!this.product.source
    },
    hasPriceOrigin() {
      return this.hasPrice && !!this.price.origins_tags && this.price.origins_tags.length
    },
    hasPriceLabels() {
      return this.hasPrice && !!this.price.labels_tags && this.price.labels_tags.length
    },
    getPriceCategoryName() {
      if (this.price && this.hasCategoryTag) {
        return utils.getCategoryName(this.price.category_tag)
      }
    },
  },
  methods: {
    getPriceProductTitle() {
      if (this.hasProductName) {
        return this.product.product_name
      } else if (this.hasPrice && this.price.product_code) {
        return this.price.product_code
      } else if (this.hasPrice && this.hasCategoryTag) {
        return this.getPriceCategoryName
      }
      return this.$t('PriceCard.UnknownProduct')
    },
    getPriceProductCode() {
      if (this.hasProduct) {
        return this.product.code
      } else if (this.price.product_code) {
        return this.price.product_code
      } else if (this.price.category_tag) {
        return this.price.category_tag
      }
      return 'product code error'
    },
    goToProduct() {
      if (this.readonly) {
        return
      }
      this.$router.push({ path: `/products/${this.getPriceProductCode()}` })
    },
  },
}
</script>
