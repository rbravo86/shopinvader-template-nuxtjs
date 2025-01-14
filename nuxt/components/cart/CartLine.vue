<template>
  <div
    class="cartline"
    :class="{ 'cartline--pending': hasPendingTransactions }"
  >
    <div class="cartline__image">
      <slot v-if="product" name="image" :images="product?.images || null">
        <product-image
          v-if="product && product?.images && product.images.length > 0"
          :image="product.images[0]"
          class="product-hit__image"
          @click="linkToProduct()"
        >
        </product-image>
      </slot>
    </div>
    <div class="cartline__content">
      <div class="content__header">
        <slot name="header" :line="line"></slot>
      </div>
      <div class="content__delete">
        <button
          type="button"
          class="btn-ghost btn-circle btn"
          :title="$t('cart.line.delete')"
          @click="deleteLine"
        >
          <Icon icon="ph:trash" class="text-xl" />
        </button>
      </div>
      <div class="content__title">
        <slot name="title" :line="line">
          <template v-if="product">
            <nuxt-link
              class="title"
              :to="localePath({ path: '/' + product.urlKey })"
            >
              {{ product.name }}
            </nuxt-link>
          </template>
        </slot>
      </div>
      <div class="content__qty">
        <slot name="qty" :line="line">
          <div class="label">
            {{ $t('cart.line.quantity') }}
          </div>
          <div class="value">
            <cart-line-qty :line="line"></cart-line-qty>
          </div>
        </slot>
      </div>
      <div class="content__price">
        <slot name="price" :line="line">
          <div class="label">
            {{ $t('cart.line.total') }}
          </div>
          <div class="value">
            <div
              v-if="line.amount.totalWithoutDiscount"
              class="price__original"
            >
              {{ $filter.currency(line.amount.totalWithoutDiscount) }}
            </div>
            <div class="price__value">
              {{ $filter.currency(line.amount.total) }}
            </div>
          </div>
        </slot>
      </div>
      <div class="content__footer">
        <slot name="footer" :line="line"></slot>
      </div>
    </div>
  </div>
</template>
<script lang="ts">
import { PropType } from 'vue'
import { CartLine } from '~/models'
import CartLineQtyVue from './CartLineQty.vue'
import ProductImageVue from '../product/ProductImage.vue'
export default {
  name: 'CartLine',
  components: {
    'product-image': ProductImageVue,
    'cart-line-qty': CartLineQtyVue
  },
  props: {
    line: {
      type: Object as PropType<CartLine>,
      required: true
    }
  },
  emits: ['deleteLine'],
  computed: {
    product() {
      return this.line?.product || false
    },
    hasPendingTransactions() {
      return this.line?.hasPendingTransactions || false
    }
  },
  methods: {
    linkToProduct() {
      if (this.product) {
        this.$router.push({
          path: '/' + this.product?.urlKey
        })
      }
    },
    deleteLine() {
      const cartService = useShopinvaderServices()?.cart || null
      if (cartService !== null) {
        cartService.deleteItem(this.line.id)
      }
      this.$emit('deleteLine', this.line)
    }
  }
}
</script>

<style lang="scss">
.cartline {
  @apply mb-2 flex flex-wrap border p-3;
  &--pending {
    .cartline__content {
      .content__price .value {
        @apply animate-pulse blur;
      }
    }
  }
  &__image {
    @apply w-32;
  }
  &__content {
    @apply flex flex-grow flex-col px-4;
    .content {
      &__header {
      }
      &__delete {
        @apply flex flex-row justify-end;
      }
      &__title {
        .title {
          @apply text-xl font-bold;
        }
      }
      &__qty {
        @apply flex flex-row justify-between py-2;
      }
      &__price {
        @apply flex flex-row justify-between py-2;
        .price {
          &__value {
            @apply pb-0 text-lg text-xl leading-3;
          }
          &__tax {
            @apply text-xs font-normal text-gray-500;
          }
          &__original {
            @apply text-sm font-normal text-gray-500 line-through;
          }
        }
      }
      &__footer {
        @apply flex flex-row justify-between;
      }
    }
  }
}
</style>
