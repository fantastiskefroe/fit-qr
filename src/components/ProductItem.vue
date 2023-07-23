<template>
  <div class="row align-items-center">
    <div class="col-2 text-center">
      <img :src="product.imgUrl" loading="lazy" width="64" :alt="product.title + 'thumbnail'">
    </div>
    <div class="col-6">
      <div class="d-flex flex-column">
        <span v-html="highlightedTitle"></span>
        <span class="small fst-italic text-muted" v-html="highlightedSkus"></span>
      </div>
    </div>
    <div class="col text-end">
      <div class="btn-group-vertical btn-group-sm" role="group" aria-label="Vertical button group">
        <button v-for="variant in product.variants" :key="variant.id"
                @click="download(product, variant)"
                type="button"
                class="btn btn-primary">
          {{ variant.title }}
        </button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import {defineComponent, PropType} from 'vue';
import QRCode from 'qrcode';
import {Product} from '@/types/product';
import {Variant} from "@/types/variant";

export default defineComponent({
  name: 'ProductItem',
  props: {
    product: {type: Object as PropType<Product>, required: true},
    highlight: String
  },
  computed: {
    highlightedTitle() {
      if (!this.highlight || this.highlight.length < 2) {
        return this.product.title;
      }

      return this.product.title.replaceAll(new RegExp(this.highlight, 'gi'), '<b>$&</b>');
    },
    highlightedSkus() {
      const skuString = this.product.variants.map(variant => variant.sku).filter(sku => sku).join(', ');
      if (!this.highlight || this.highlight.length < 2) {
        return skuString;
      }

      return skuString.replaceAll(new RegExp(this.highlight, 'gi'), '<b>$&</b>');
    }
  }, methods: {
    async download(product: Product, variant: Variant): Promise<void> {
      return this.downloadQrWithInput(
          `${product.url}?variant=${variant.id}`,
          this.generateFileName(product, variant)
      );
    },
    generateFileName(product: Product, variant: Variant): string {
      return `${variant.sku}-qr.svg`;
    },
    async downloadQrWithInput(inputString: string, fileName: string): Promise<void> {
      if (!inputString) {
        return;
      }

      const objectUrl = this.generateObjectURL(await this.qr(inputString, 45.354));

      // Create and activate link element
      const el = document.createElement('a');
      el.download = fileName;
      el.href = objectUrl;
      el.click();

      // Clean up
      el.remove();
      window.URL.revokeObjectURL(objectUrl);
    },

    async qr(inputString: string, width?: number): Promise<string> {
      return QRCode.toString(inputString, {
        type: 'svg',
        errorCorrectionLevel: 'H',
        margin: 0,
        color: {
          dark: '#000000FF',
          light: '#FFFFFF00'
        },
        width
      });
    },
    generateObjectURL(data: string): string {
      return window.URL.createObjectURL(new Blob([data]));
    },
  }
});
</script>

<style scoped>

</style>