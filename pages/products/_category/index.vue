<template>
  <div>
    <b-breadcrumb :items="breadcrumbItems"></b-breadcrumb>
    <template v-if="category">
      <h1>{{ category.name }}</h1>
      <p>{{ category.description }}</p>
    </template>
    <div class="product-list__toolbar">
      <b-button-group>
        <b-button
          v-for="option in sortingOptions"
          :key="option.label"
          variant="outline-secondary"
          :pressed="selectedSortingOption === option"
          @click="selectedSortingOption = option"
        >
          {{ option.label }}
        </b-button>
      </b-button-group>
    </div>
    <b-row>
      <b-col
        v-for="product in products"
        :key="product.id"
        cols="12"
        sm="6"
        md="4"
        lg="3"
        class="product-list__tile"
      >
        <product-tile :product="product" />
      </b-col>
    </b-row>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Watch } from 'nuxt-property-decorator'
import { IContentDocument } from '@nuxt/content/types/content'
import { CategoryDocument } from '@/types/categories/category'
import { ProductDocument } from '@/types/products/product'

type SortingOption = { label: string; field: string; direction: 'asc' | 'desc' }
const sortingOptions: SortingOption[] = [
  { label: 'Sort by name ascending', field: 'name', direction: 'asc' },
  { label: 'Sort by name descending', field: 'name', direction: 'desc' },
]

const rootBreadcrumbItem = { text: 'All Products', to: '/products' }

@Component({
  async asyncData({ $content, params }) {
    const [category, products] = await Promise.all([
      $content('categories', params.category).fetch<CategoryDocument>() as Promise<CategoryDocument & IContentDocument>,
      fetchProducts($content, params.category, sortingOptions[0]),
    ])
    const breadcrumbItems = [
      rootBreadcrumbItem,
      { text: category.name, active: true },
    ]
    return { category, products, breadcrumbItems }
  },
})
export default class ProductCategoryPage extends Vue {
  category: (CategoryDocument & IContentDocument) | null = null
  products: Partial<ProductDocument & IContentDocument>[] = []
  selectedSortingOption: SortingOption = sortingOptions[0]
  breadcrumbItems = []

  get sortingOptions(): SortingOption[] {
    return sortingOptions
  }

  head() {
    return {
      title: this.category ? this.category.name : 'Category',
      meta: [
        {
          hid: 'description',
          name: 'description',
          content: `Overview of products belonging to category ${this.category?.name}`,
        },
      ],
    }
  }

  @Watch('selectedSortingOption')
  async onSortingChanged(sortingOption: SortingOption) {
    this.products = await fetchProducts(
      this.$content,
      this.$route.params.category,
      sortingOption
    )
  }
}

function fetchProducts(
  $content: Vue['$content'],
  categorySlug: string,
  sortingOption: SortingOption
): Promise<Partial<ProductDocument & IContentDocument>[]> {
  return $content('products', categorySlug)
    .only(['id', 'name', 'description', 'slug', 'category'])
    .sortBy(sortingOption.field, sortingOption.direction)
    .fetch<ProductDocument>() as Promise<Partial<ProductDocument & IContentDocument>[]>
}
</script>

<style lang="scss" scoped>
@import '@/assets/css/variables';

.product-list__toolbar {
  margin-bottom: $grid-gutter-width;
}
.product-list__tile {
  margin-bottom: $grid-gutter-width;
}
</style>
