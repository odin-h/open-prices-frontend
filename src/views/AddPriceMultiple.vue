<template>
  <h1 class="text-h5 mb-1">
    {{ $t('AddPriceMultiple.Title') }}
  </h1>

  <v-row>
    <!-- Step 1: proof -->
    <v-col cols="12" md="6" lg="4">
      <v-card
        :title="(proofType === 'RECEIPT') ? $t('AddPriceHome.ReceiptMode.Title') : $t('AddPriceMultiple.ProofDetails.Title')"
        :prepend-icon="(proofType === 'RECEIPT') ? 'mdi-receipt-text-outline' : 'mdi-library-shelves'"
        height="100%"
        :style="proofFormFilled ? 'border: 1px solid #4CAF50' : 'border: 1px solid transparent'">
        <template v-slot:append v-if="proofFormFilled">
          <v-icon icon="mdi-checkbox-marked-circle" color="success"></v-icon>
        </template>
        <v-divider></v-divider>
        <v-card-text>
          <v-row>
            <v-col cols="8">
              <v-btn class="mb-2 mr-2" size="small" prepend-icon="mdi-camera" @click.prevent="$refs.proofCamera.click()" :loading="createProofLoading" :disabled="createProofLoading">
                <span class="d-sm-none">{{ $t('AddPriceSingle.PriceDetails.Picture') }}</span>
                <span class="d-none d-sm-inline-flex">{{ $t('AddPriceSingle.PriceDetails.TakePicture') }}</span>
              </v-btn>
              <v-btn class="mb-2 mr-2" size="small" prepend-icon="mdi-image-plus" @click.prevent="$refs.proofGallery.click()" :loading="createProofLoading" :disabled="createProofLoading">
                <span class="d-sm-none">{{ $t('AddPriceSingle.PriceDetails.Gallery') }}</span>
                <span class="d-none d-sm-inline-flex">{{ $t('AddPriceSingle.PriceDetails.SelectFromGallery') }}</span>
              </v-btn>
              <v-btn class="mb-2" size="small" prepend-icon="mdi-receipt-text-clock" @click="showUserRecentProofs">
                <span class="d-sm-none">{{ $t('AddPriceSingle.PriceDetails.RecentProof') }}</span>
                <span class="d-none d-sm-inline-flex">{{ $t('AddPriceSingle.PriceDetails.SelectRecentProof') }}</span>
              </v-btn>
              <v-file-input
                class="d-none overflow-hidden"
                ref="proofCamera"
                v-model="proofImage"
                capture="environment"
                accept="image/*"
                @change="newProof('camera')"
                @click:clear="clearProof"
                :loading="createProofLoading">
              </v-file-input>
              <v-file-input
                class="d-none overflow-hidden"
                ref="proofGallery"
                v-model="proofImage"
                accept="image/*, .heic"
                @change="newProof('gallery')"
                @click:clear="clearProof"
                :loading="createProofLoading">
              </v-file-input>
              <p v-if="proofFormFilled && !createProofLoading" class="text-green mt-2 mb-2">
                <i v-if="!proofisSelected">{{ $t('AddPriceSingle.PriceDetails.ProofUploaded') }}</i>
                <i v-if="proofisSelected">{{ $t('AddPriceSingle.PriceDetails.ProofSelected') }}</i>
              </p>
              <p v-if="!proofFormFilled && !createProofLoading" class="text-red mt-2 mb-2">
                <i>{{ $t('AddPriceSingle.PriceDetails.UploadProof') }}</i>
              </p>
            </v-col>
            <v-col cols="4" v-if="proofFormFilled">
              <v-img :src="proofImagePreview" style="max-height:200px"></v-img>
            </v-col>
          </v-row>
          <v-row v-if="proofType === 'RECEIPT'" class="mt-0">
            <v-col>
              <h3 class="mb-1">{{ $t('ProofDetail.Privacy') }}</h3>
              <p class="text-caption text-warning">
                <i>{{ $t('AddPriceMultiple.ProofDetails.ReceiptWarning') }}</i>
              </p>
            </v-col>
          </v-row>
        </v-card-text>
        <v-overlay v-model="disableProofLocationDateForm" scrim="#E8F5E9" contained persistent></v-overlay>
      </v-card>
    </v-col>

    <!-- Step 2: location & date -->
    <v-col cols="12" md="6" lg="4">
      <v-card
        :title="$t('AddPriceSingle.WhereWhen.Title')"
        prepend-icon="mdi-map-marker-outline"
        height="100%"
        :style="locationDateFormFilled ? 'border: 1px solid #4CAF50' : 'border: 1px solid transparent'">
        <template v-slot:append v-if="locationDateFormFilled">
          <v-icon icon="mdi-checkbox-marked-circle" color="success"></v-icon>
        </template>
        <v-divider></v-divider>
        <v-card-text>
          <h3 class="mb-1">{{ $t('AddPriceSingle.WhereWhen.Location') }}</h3>
          <v-chip
            class="mb-2"
            :style="isSelectedLocation(location) ? 'border: 1px solid #4CAF50' : 'border: 1px solid transparent'"
            v-for="location in recentLocations"
            :key="getLocationUniqueID(location)"
            @click="setLocationData(location)">
            <v-icon start :icon="isSelectedLocation(location) ? 'mdi-check-circle-outline' : 'mdi-history'" :color="isSelectedLocation(location) ? 'green' : ''"></v-icon>
            {{ getLocationTitle(location, true, true, true) }}
          </v-chip>
          <br v-if="recentLocations.length" />
          <v-btn class="mb-2" size="small" prepend-icon="mdi-magnify" @click="showLocationSelectorDialog">{{ $t('AddPriceSingle.WhereWhen.Find') }}</v-btn>
          <p v-if="!locationFormFilled" class="text-red mb-2"><i>{{ $t('AddPriceSingle.WhereWhen.SelectLocation') }}</i></p>

          <h3 class="mt-4 mb-1">{{ $t('AddPriceSingle.WhereWhen.Date') }}</h3>
          <v-row>
            <v-col cols="12" sm="6">
              <v-text-field
                v-model="addPriceMultipleForm.date"
                :label="$t('AddPriceSingle.WhereWhen.DateLabel')"
                type="date"
              ></v-text-field>
            </v-col>
          </v-row>
        </v-card-text>
        <v-overlay v-model="disableProofLocationDateForm" scrim="#E8F5E9" contained persistent></v-overlay>
      </v-card>
    </v-col>

    <!-- Step 3a: product prices already uploaded -->
    <v-col v-if="productPriceUploadedList.length" cols="12" md="6" lg="4">
      <v-card
        prepend-icon="mdi-tag-check-outline"
        height="100%"
        style="border: 1px solid #4CAF50">
        <template v-slot:title>
          <i18n-t keypath="AddPriceMultiple.ProductPriceDetails.AlreadyUploaded" :plural="productPriceUploadedList.length" tag="span">
            <template v-slot:priceAlreadyUploadedNumber>
              <span>{{ productPriceUploadedList.length }}</span>
            </template>
          </i18n-t>
        </template>
        <template v-slot:append>
          <v-icon icon="mdi-checkbox-marked-circle" color="success"></v-icon>
        </template>
        <v-divider></v-divider>
        <v-card-text>
          <v-row>
            <v-col v-for="(productPriceUploaded, index) in productPriceUploadedList" cols="12" :key="productPriceUploaded">
              <PriceCard :price="productPriceUploaded" :product="productPriceUploaded.product" :hidePriceDate="true" :hidePriceFooterRow="true" :readonly="true"></PriceCard>
            </v-col>
          </v-row>
        </v-card-text>
        <v-overlay v-model="disablePriceAlreadyUploadedCard" scrim="#E8F5E9" contained persistent></v-overlay>
      </v-card>
    </v-col>

    <!-- Step 3b: new product price form -->
    <v-col v-if="Object.keys(productPriceForm).length" cols="12" md="6" lg="4">
      <v-card
        :title="$t('AddPriceMultiple.ProductPriceDetails.NewPrice')"
        prepend-icon="mdi-tag-outline"
        append-icon="mdi-delete"
        height="100%"
        style="border: 1px solid transparent">
        <template v-slot:append>
          <v-icon icon="mdi-delete" @click="clearProductPriceForm" color="error"></v-icon>
        </template>
        <v-divider></v-divider>
        <v-card-text>
          <h3 class="mb-2">
            <v-item-group v-model="productMode" class="d-inline" mandatory>
              <v-item v-for="pm in productModeList" :key="pm.key" :value="pm.key" v-slot="{ isSelected, toggle }">
                <v-chip class="mr-1" @click="toggle" :style="isSelected ? 'border: 1px solid #9E9E9E' : 'border: 1px solid transparent'">
                  <v-icon start :icon="isSelected ? 'mdi-checkbox-marked-circle' : 'mdi-circle-outline'"></v-icon>
                  {{ pm.value }}
                </v-chip>
              </v-item>
            </v-item-group>
          </h3>
          <v-sheet v-if="productMode === 'barcode'">
            <v-btn class="mb-2 mr-2" size="small" prepend-icon="mdi-barcode-scan" @click="showBarcodeScannerDialog">{{ $t('AddPriceSingle.ProductInfo.ScanBarcode') }}</v-btn>
            <a href="#" @click.prevent="showBarcodeManualInputDialog">{{ $t('AddPriceSingle.ProductInfo.TypeBarcode') }}</a>
            <v-text-field
              v-if="dev"
              :prepend-inner-icon="productBarcodeFormFilled ? 'mdi-barcode' : 'mdi-barcode-scan'"
              v-model="productPriceForm.product_code"
              :label="$t('AddPriceSingle.ProductInfo.ProductBarcode')"
              type="text"
              hint="EAN"
              hide-details="auto"
              @click:prepend="showBarcodeScannerDialog"
            ></v-text-field>
            <ProductCard v-if="product" class="mb-4" :product="product" :readonly="true" elevation="1"></ProductCard>
          </v-sheet>
          <v-sheet v-if="productMode === 'category'">
            <v-row>
              <v-col cols="6">
                <v-autocomplete
                  :prepend-inner-icon="productCategoryFormFilled ? 'mdi-basket-check-outline' : 'mdi-basket-outline'"
                  v-model="productPriceForm.category_tag"
                  :label="$t('AddPriceSingle.ProductInfo.CategoryLabel')"
                  :items="categoryTags"
                  :item-title="item => item.name"
                  :item-value="item => item.id"
                  hide-details="auto"
                ></v-autocomplete>
              </v-col>
              <v-col cols="6">
                <v-autocomplete
                  v-model="productPriceForm.origins_tags"
                  :label="$t('AddPriceSingle.ProductInfo.OriginLabel')"
                  :items="originTags"
                  :item-title="item => item.name"
                  :item-value="item => item.id"
                  hide-details="auto"
                ></v-autocomplete>
              </v-col>
            </v-row>
            <div class="d-inline">
              <v-checkbox
                v-for="lt in labelsTags"
                v-model="productPriceForm.labels_tags"
                :label="lt.name"
                :value="lt.id"
                hide-details="auto"
              ></v-checkbox>
            </div>
          </v-sheet>
          <p v-if="!productFormFilled" class="text-red mt-2 mb-2">
            <i>{{ $t('AddPriceSingle.ProductInfo.SetProduct') }}</i>
          </p>
          <h3 class="mb-1">
            <v-item-group v-if="productMode === 'category'" v-model="productPriceForm.price_per" class="d-inline" mandatory>
                <v-item v-for="cpp in categoryPricePerList" :key="cpp.key" :value="cpp.key" v-slot="{ isSelected, toggle }">
                  <v-chip class="mr-1" @click="toggle" :style="isSelected ? 'border: 1px solid #9E9E9E' : 'border: 1px solid transparent'">
                    <v-icon start :icon="isSelected ? 'mdi-checkbox-marked-circle' : 'mdi-circle-outline'"></v-icon>
                    {{ cpp.value }}
                  </v-chip>
                </v-item>
            </v-item-group>
          </h3>
          <PriceInputRow :priceForm="productPriceForm"></PriceInputRow>
        </v-card-text>
        <v-divider></v-divider>
        <v-card-text>
          <v-btn
            color="success"
            :loading="createPriceLoading"
            :disabled="!productPriceFormFilled"
            @click="createPrice"
          >{{ $t('AddPriceMultiple.ProductPriceDetails.Upload') }}</v-btn>
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>

  <v-row>
    <v-col>
      <v-btn
        v-if="!Object.keys(productPriceForm).length"
        class="mr-2"
        prepend-icon="mdi-plus"
        color="primary"
        :loading="createPriceLoading"
        :disabled="!proofLocationFormFilled || !productFormFilled"
        @click="initNewProductPriceForm"
      >{{ $t('AddPriceMultiple.ProductPriceDetails.Add') }}</v-btn>
      <v-btn
        class="float-right"
        type="submit"
        :loading="createPriceLoading"
        :disabled="!proofLocationFormFilled || !productPriceUploadedList.length"
        @click="done"
      >{{ $t('AddPriceMultiple.Done') }}</v-btn>
    </v-col>
  </v-row>

  <v-snackbar
    v-model="proofDateSuccessMessage"
    color="info"
    :timeout="2000"
  >{{ $t('AddPriceSingle.PriceDetails.ProofDateChanged') }}</v-snackbar>
  <v-snackbar
    v-model="proofSuccessMessage"
    color="success"
    :timeout="2000"
  >{{ $t('AddPriceSingle.PriceDetails.ProofUploaded') }}</v-snackbar>
  <v-snackbar
    v-model="proofSelectedSuccessMessage"
    color="success"
    :timeout="2000"
  >{{ $t('AddPriceSingle.PriceDetails.ProofSelected') }}</v-snackbar>
  <v-snackbar
    v-model="priceSuccessMessage"
    color="success"
    :timeout="2000"
  >{{ $t('AddPriceMultiple.ProductPriceDetails.PriceUploaded') }}</v-snackbar>

  <LocationSelectorDialog
    v-if="locationSelectorDialog"
    v-model="locationSelectorDialog"
    @location="setLocationData($event)"
    @close="locationSelectorDialog = false"
  ></LocationSelectorDialog>
  <BarcodeScannerDialog
    v-if="barcodeScannerDialog"
    v-model="barcodeScannerDialog"
    @barcode="setProductCode($event)"
    @close="barcodeScannerDialog = false"
  ></BarcodeScannerDialog>
  <BarcodeManualInputDialog
    v-if="barcodeManualInputDialog"
    v-model="barcodeManualInputDialog"
    @barcode="setProductCode($event)"
    @close="barcodeManualInputDialog = false"
  ></BarcodeManualInputDialog>
  <UserRecentProofsDialog
    v-if="userRecentProofsDialog"
    v-model="userRecentProofsDialog"
    @recentProofSelected="handleRecentProofSelected($event)"
    @close="userRecentProofsDialog = false"
  ></UserRecentProofsDialog>
</template>

<script>
import Compressor from 'compressorjs'
import ExifReader from 'exifreader'
import { defineAsyncComponent } from 'vue'
import { mapStores } from 'pinia'
import { useAppStore } from '../store'
import api from '../services/api'
import utils from '../utils.js'
import LabelsTags from '../data/labels-tags.json'

Compressor.setDefaults({
  checkOrientation: true,  // default
  retainExif: true,
  mimeType: 'image/webp'
})

export default {
  components: {
    'LocationSelectorDialog': defineAsyncComponent(() => import('../components/LocationSelectorDialog.vue')),
    'PriceInputRow': defineAsyncComponent(() => import('../components/PriceInputRow.vue')),
    'PriceCard': defineAsyncComponent(() => import('../components/PriceCard.vue')),
    'ProductCard': defineAsyncComponent(() => import('../components/ProductCard.vue')),
    'BarcodeScannerDialog': defineAsyncComponent(() => import('../components/BarcodeScannerDialog.vue')),
    'BarcodeManualInputDialog': defineAsyncComponent(() => import('../components/BarcodeManualInputDialog.vue')),
    'UserRecentProofsDialog': defineAsyncComponent(() => import('../components/UserRecentProofsDialog.vue')),
  },
  data() {
    return {
      dev: import.meta.env.DEV,
      proofType: null,  // 'PRICE_TAG' or 'RECEIPT'
      // price form
      addPriceMultipleForm: {
        proof_id: null,
        location_osm_id: null,
        location_osm_type: '',
        date: utils.currentDate(),
      },
      productPriceForm: {},
      createPriceLoading: false,
      priceSuccessMessage: false,
      // proof data
      proofImage: null,
      proofImagePreview: null,
      createProofLoading: false,
      proofDateSuccessMessage: false,
      proofSuccessMessage: false,
      userRecentProofsDialog: false,
      proofSelectedSuccessMessage: false,
      proofisSelected: false,
      // location data
      locationSelectorDialog: false,
      // product price data
      productPriceUploadedList: [],
      productPriceNew: {
        product_code: '',
        category_tag: null,
        origins_tags: '',
        labels_tags: [],
        price: null,
        price_per: null, // see initPriceSingleForm
        price_is_discounted: false,
        price_without_discount: null,
        currency: null,  // see initPriceMultipleForm
        uploaded: false
      },
      product: null,
      productModeList: [
        {key: 'barcode', value: this.$t('AddPriceSingle.ProductModeList.Barcode'), icon: 'mdi-barcode-scan'},
        {key: 'category', value: this.$t('AddPriceSingle.ProductModeList.Category'), icon: 'mdi-basket-outline'}
      ],
      productMode: null,
      categoryTags: [],  // list of category tags for autocomplete  // see initPriceMultipleForm
      originTags: [],  // list of origins tags for autocomplete  // see initPriceMultipleForm
      labelsTags: LabelsTags,
      barcodeScannerDialog: false,
      barcodeManualInputDialog: false,
      categoryPricePerList: [
        {key: 'KILOGRAM', value: this.$t('AddPriceSingle.CategoryPricePer.PerKg'), icon: 'mdi-weight-kilogram'},
        {key: 'UNIT', value: this.$t('AddPriceSingle.CategoryPricePer.PerUnit'), icon: 'mdi-numeric-1-circle'}
      ],
     }
  },
  computed: {
    ...mapStores(useAppStore),
    proofFormFilled() {
      let keys = ['proof_id']
      return Object.keys(this.addPriceMultipleForm).filter(k => keys.includes(k)).every(k => !!this.addPriceMultipleForm[k])
    },
    recentLocations() {
      return this.appStore.getRecentLocations(3)
    },
    locationFormFilled() {
      let keys = ['location_osm_id', 'location_osm_type']
      return Object.keys(this.addPriceMultipleForm).filter(k => keys.includes(k)).every(k => !!this.addPriceMultipleForm[k])
    },
    locationDateFormFilled() {
      let keys = ['location_osm_id', 'location_osm_type', 'date']
      return Object.keys(this.addPriceMultipleForm).filter(k => keys.includes(k)).every(k => !!this.addPriceMultipleForm[k])
    },
    proofLocationFormFilled() {
      return this.proofFormFilled && this.locationDateFormFilled
    },
    productBarcodeFormFilled() {
      let keys = ['product_code']
      return Object.keys(this.productPriceForm).filter(k => keys.includes(k)).every(k => !!this.productPriceForm[k])
    },
    productCategoryFormFilled() {
      let keys = ['category_tag', 'origins_tags']
      return Object.keys(this.productPriceForm).filter(k => keys.includes(k)).every(k => !!this.productPriceForm[k])
    },
    productFormFilled() {
      return this.productBarcodeFormFilled || this.productCategoryFormFilled
    },
    priceFormFilled() {
      let keys = ['price', 'currency']
      return Object.keys(this.productPriceForm).filter(k => keys.includes(k)).every(k => !!this.productPriceForm[k])
    },
    productPriceFormFilled() {
      return this.productFormFilled && this.priceFormFilled
    },
    formFilled() {
      return this.proofLocationFormFilled && !!this.productPriceUploadedList.length && !Object.keys(this.productPriceForm).length
    },
    disableProofLocationDateForm() {
      return this.proofLocationFormFilled && !!this.productPriceUploadedList.length
    },
    disablePriceAlreadyUploadedCard() {
      return !!this.productPriceUploadedList.length
    }
  },
  mounted() {
    this.initPriceMultipleForm()
  },
  methods: {
    initPriceMultipleForm() {
      /**
       * init form config (product mode, categories, origins, last locations)
       * (init form done in initNewProductPriceForm)
       */
      this.proofType = this.$route.path.endsWith('/receipt') ? 'RECEIPT' : 'PRICE_TAG'
      utils.getLocaleCategoryTags(this.appStore.getUserLanguage).then((module) => {
        this.categoryTags = module.default
      })
      utils.getLocaleOriginTags(this.appStore.getUserLanguage).then((module) => {
        this.originTags = module.default
      })
      if (this.recentLocations.length) {
        this.setLocationData(this.recentLocations[0])
      }
    },
    addPriceToUploadedList(price) {
      this.productPriceUploadedList.push(price)
    },
    showUserRecentProofs() {
      this.userRecentProofsDialog = true
    },
    handleRecentProofSelected(selectedProof) {
      this.addPriceMultipleForm.proof_id = selectedProof.id
      this.proofImagePreview = this.getProofUrl(selectedProof)
      this.proofSelectedSuccessMessage = true
      this.proofisSelected = true
    },
    getProofUrl(proof) {
      // return 'https://prices.openfoodfacts.org/img/0002/qU59gK8PQw.webp'
      return `${import.meta.env.VITE_OPEN_PRICES_APP_URL}/img/${proof.file_path}`
    },
    newProof(source) {
      if (source === 'gallery') {
        ExifReader.load(this.proofImage[0]).then((tags) => {
          if (tags['DateTimeOriginal'] && tags['DateTimeOriginal'].description) {
            // exif DateTimeOriginal format: '2024:01:31 20:23:52'
            const imageDateString = tags['DateTimeOriginal'].description.substring(0, 10).replaceAll(':', '-')
            if (imageDateString !== this.addPriceMultipleForm.date) {
              this.addPriceMultipleForm.date = imageDateString
              this.proofDateSuccessMessage = true
            }
          }
        })
      }
      this.uploadProof()
    },
    uploadProof() {
      this.createProofLoading = true
      new Promise((resolve, reject) => {
        new Compressor(this.proofImage[0], {
          success: resolve,
          error: reject
        })
      })
      .then((proofImageCompressed) => {
        api
          .createProof(proofImageCompressed, this.proofType)
          .then((data) => {
            this.createProofLoading = false
            if (data['id']) {
              const store = useAppStore()
              store.addProof(data)
              this.addPriceMultipleForm.proof_id = data['id']
              this.proofImagePreview = URL.createObjectURL(proofImageCompressed)
              this.proofSuccessMessage = true
            } else {
              alert('Error: server error')
              console.log(data)
            }
          })
          .catch((error) => {
            alert('Error: server error')
            console.log(error)
            this.createProofLoading = false
          })
      })
      .catch((error) => {
        alert('Error: compression')
        console.log(error)
      })
      // .finally(() => {
      //   console.log('Compress complete')
      // })
    },
    clearProof() {
      this.proofImage = null
      this.proofImagePreview = null
      this.addPriceMultipleForm.proof_id = null
    },
    showLocationSelectorDialog() {
      this.locationSelectorDialog = true
    },
    getLocationTitle(location, withName=true, withRoad=false, withCity=true) {
      return utils.getLocationTitle(location, withName, withRoad, withCity)
    },
    getLocationUniqueID(location) {
      return utils.getLocationUniqueID(location)
    },
    setLocationData(location) {
      this.appStore.addRecentLocation(location)
      this.addPriceMultipleForm.location_osm_id = utils.getLocationID(location)
      this.addPriceMultipleForm.location_osm_type = utils.getLocationType(location)
    },
    isSelectedLocation(location) {
      return (this.addPriceMultipleForm.location_osm_id === utils.getLocationID(location)) && (this.addPriceMultipleForm.location_osm_type === utils.getLocationType(location))
    },
    showBarcodeScannerDialog() {
      this.barcodeScannerDialog = true
    },
    showBarcodeManualInputDialog() {
      this.barcodeManualInputDialog = true
    },
    setProductCode(code) {
      this.productPriceForm.product_code = code
      this.product = null
      api
        .getProductByCode(code)
        .then((data) => {
          this.product = data.id ? data : {'code': code}
          console.log(this.product)
        })
        .catch((error) => {
          alert("Error: Open Prices server error")
        })
    },
    clearProductPriceForm() {
      this.productPriceForm = {}
      this.product = null
    },
    initNewProductPriceForm() {
      this.productMode = this.appStore.user.last_product_mode_used
      this.clearProductPriceForm()
      this.productPriceForm = JSON.parse(JSON.stringify(this.productPriceNew))
      this.productPriceForm.currency = this.appStore.getUserLastCurrencyUsed
      this.productPriceForm.price_per = this.categoryPricePerList[0].key // init to 'KILOGRAM' because it's the most common use-case
    },
    createPrice() {
      this.createPriceLoading = true
      this.appStore.setLastCurrencyUsed(this.productPriceForm.currency)
      // cleanup form
      if (!this.productPriceForm.product_code) {
        this.productPriceForm.product_code = null
      } else {
        this.productPriceForm.price_per = null
      }
      if ((typeof this.productPriceForm.origins_tags === 'string') && (this.productPriceForm.origins_tags.length)) {
        this.productPriceForm.origins_tags = [this.productPriceForm.origins_tags]
      } else {
        this.productPriceForm.origins_tags = null
      }
      if (this.productPriceForm.labels_tags.length == 0) {
        this.productPriceForm.labels_tags = null
      }
      if (!this.productPriceForm.price_is_discounted) {
        this.productPriceForm.price_without_discount = null
      }
      // create price
      api
        .createPrice(Object.assign({}, this.addPriceMultipleForm, this.productPriceForm))
        .then((data) => {
          if (data['detail']) {
            alert(`Error: with input ${data['detail'][0]['input']}`)
          } else {
            this.addPriceToUploadedList(Object.assign({}, this.productPriceForm, {product: this.product}))
            this.priceSuccessMessage = true
            // show new price form immediately
            this.initNewProductPriceForm()
          }
          this.createPriceLoading = false
        })
        .catch((error) => {
          alert('Error: server error')
          console.log(error)
          this.createPriceLoading = false
        })
    },
    done() {
      this.$router.push({ path: '/add', query: { multipleSuccess: 'true' } })
    }
  },
  watch: {
    productMode(newProductMode, oldProductMode) {
      // reset product_code and category_tag when switching mode
      if (oldProductMode) {
        this.productPriceForm.product_code = ""
        this.productPriceForm.category_tag = null
        this.productPriceForm.origins_tags = ''
        this.productPriceForm.labels_tags = []
        this.product = null
      }
    }
  }
}
</script>
