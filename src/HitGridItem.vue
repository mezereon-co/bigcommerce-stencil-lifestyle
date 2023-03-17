<template>
  <div class="product">
    <slot>
      <article class="card">
        <div class="card-inner">
          <figure class="card-figure">
            <template v-if="isOnSale">
              <template v-if="context.productSaleBadges === 'sash'">
                <div class="sale-flag-sash">
                  <span class="sale-text">Sale</span>
                </div>
              </template>
              <template v-if="context.productSaleBadges === 'topleft'">
                <div class="sale-flag-side">
                  <span class="sale-text">Sale</span>
                </div>
              </template>
              <template v-if="context.productSaleBadges === 'burst'">
                <div class="starwrap">
                  <div class="sale-text-burst">Sale</div>
                  <div class="sale-flag-star"></div>
                </div>
              </template>
            </template>
            <a
              :href="item.custom_url.url"
              @click="trackEvent(item, index, 'click')"
            >
              <div class="card-img-container show-image">
                <img
                  v-lazy="{
                    src: item.image,
                    error: 'https://via.placeholder.com/338/eee'
                  }"
                  class="card-image"
                />
              </div>
            </a>
            <div class="card-figcaption-body-custom">
              <a
                v-if="context.showProductQuickView"
                class="button button--small card-figcaption-button quickview"
                :data-product-id="item.id"
                @click="trackEvent(item, index, 'click')"
              >
                <!-- {{ context.langQuickView }} -->
                <svg><use xlink:href="#icon-quickview"></use></svg>
              </a>
              <a
                v-if="context.showWishlist"
                :href="'/wishlist.php?action=add&amp;product_id=' + item.id"
                class="button button--small card-figcaption-button wishlist"
                @click="trackEvent(item, index, 'add2list')"
              >
                <svg><use xlink:href="#icon-wishlist"></use></svg>
              </a>
              <div>
                <a
                  v-if="context.showCompare"
                  data-tooltip
                  class="button button--small card-figcaption-button compare-box card-button compare"
                  :class="{ 'compare-active': isCompare }"
                  :for="'compare-' + item.id"
                  :value="item.id"
                  :data-compare-id="item.id"
                >
                  <svg><use xlink:href="#icon-compare"></use></svg>
                </a>
              </div>
            </div>
          </figure>
          <div class="card-body">
            <div class="card-body-inn">
              <p
                v-if="item.brand"
                class="card-text brand-name"
                data-test-info-type="brandName"
              >
                {{ item.brand | value }}
              </p>
              <div class="card-mid-block">
                <div class="card-button-block">
                  <template v-if="context.customer || !context.restrictToLogin">
                    <a
                      v-if="item.option_set_id"
                      :href="item.custom_url.url"
                      data-event-type="product-click"
                      class="button button--small card-figcaption-button"
                      :data-product-id="item.id"
                      @click="trackEvent(item, index, 'click')"
                    >
                      <svg><use xlink:href="#icon-choose-options"></use></svg>
                      <span>{{ context.langChooseOptions }}</span>
                    </a>
                    <a
                      v-else-if="item.availability === 'preorder'"
                      :href="'/cart.php?action=add&amp;product_id=' + item.id"
                      data-event-type="product-click"
                      class="button button--small card-figcaption-button"
                      :data-product-id="item.id"
                      @click="trackEvent(item, index, 'add2cart')"
                    >
                      <svg><use xlink:href="#icon-pre-order"></use></svg>
                      <span>{{ context.langPreOrder }}</span>
                    </a>
                    <a
                      v-else-if="item.out_of_stock_message"
                      :href="item.custom_url.url"
                      data-event-type="product-click"
                      class="button button--small card-figcaption-button"
                      :data-product-id="item.id"
                      @click="trackEvent(item, index, 'click')"
                    >
                      <svg><use xlink:href="#icon-out-of-stock"></use></svg>
                      <span>{{ item.out_of_stock_message }}</span>
                    </a>
                    <span
                      v-else-if="
                        item.inventory_tracking === 'product' &&
                        item.inventory_level === 0
                      "
                    ></span>
                    <a
                      v-else
                      :href="'/cart.php?action=add&amp;product_id=' + item.id"
                      data-event-type="product-click"
                      class="button button--small card-figcaption-button"
                      @click="trackEvent(item, index, 'add2cart')"
                    >
                      <svg><use xlink:href="#icon-add-to-cart"></use></svg>
                      <span>{{ context.langAddToCart }}</span>
                    </a>
                  </template>
                </div>
                <h4 class="card-title">
                  <a
                    :href="item.custom_url.url"
                    @click="trackEvent(item, index, 'click')"
                  >
                    <span v-html="item.name"></span>
                  </a>
                </h4>
              </div>
              <div class="card-text price-block" data-test-info-type="price">
                <p v-if="!context.customer && context.restrictToLogin">
                  Log in for pricing
                </p>
                <div
                  v-else-if="item.retail_price"
                  class="price-section price-section--withoutTax rrp-price--withoutTax"
                >
                  {{ context.pdpRetailPriceLabel }}
                  <span
                    data-product-rrp-price-without-tax
                    class="price price--rrp"
                  >
                    {{ item.retail_price | currency }}
                  </span>
                </div>
                <div
                  v-if="isOnSale"
                  class="price-section price-section--withoutTax non-sale-price--withoutTax"
                  data-test-info-type="price"
                >
                  {{ context.pdpNonSalePriceLabel }}
                  <span
                    data-product-non-sale-price-without-tax
                    class="price price--non-sale"
                  >
                    {{ item.price | currency }}
                  </span>
                </div>
                <div class="price-section price-section--withoutTax">
                  <span v-if="isOnSale" class="price-now-label">
                    {{ context.pdpSalePriceLabel }}
                  </span>
                  <span v-else class="price-label">
                    {{ context.pdpPriceLabel }}
                  </span>
                  <span
                    data-product-price-without-tax
                    class="price price--withoutTax"
                  >
                    {{ item.calculated_price | currency }}
                  </span>
                </div>
              </div>
              <p class="card-text" data-test-info-type="productRating">
                <span class="rating--small">
                  <rating :rating="item.reviews_rating_sum"></rating>
                </span>
              </p>
              <p class="card-summary">
                <span
                  v-line-clamp="2"
                  v-html="this.$options.filters.striphtml(item.description)"
                ></span>
              </p>
            </div>
          </div>
        </div>
      </article>
    </slot>
  </div>
</template>

<script>
import Hits from './Hits.js'
import Rating from './Rating.vue'

export default {
  components: {
    Rating
  },
  mixins: [Hits],
  props: {
    item: {
      type: Object,
      required: true
    },
    score: {
      type: Number,
      required: true
    },
    index: {
      type: Number,
      required: true
    }
  },
  computed: {
    isOnSale() {
      return this.item.price !== this.item.calculated_price
    },
    isCompare() {
      let compareList = this.$cookies.get('compare_list')
      if (!compareList) {
        return false
      }
      if (compareList === null || compareList === '') {
        return false
      }
      // ensure string type
      compareList += ''

      // eslint-disable-next-line eqeqeq
      return compareList.split(',').find((x) => x == this.item.id)
    }
  }
}
</script>

<style></style>
