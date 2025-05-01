<script setup lang="ts">
import { getPaginationRowModel, getSortedRowModel } from '@tanstack/vue-table'
import type { TableColumn } from '@nuxt/ui'
import { h, resolveComponent } from 'vue'

definePageMeta({ title: 'Список продуктів' })
useHead({ title: 'Список продуктів' })
const table = useTemplateRef('table')
const UBadgeAlt = resolveComponent('UBadgeAlt')

type ProductResponse = {
  products: Product[]
}
type Product = {
  title: string
  description: string
  price: number
  rating: number
  brand: string
  category: string
  thumbnail: string
}

const { data, status } = await useFetch('https://dummyjson.com/products', {
  key: 'table-products',
  transform: (response: ProductResponse): Product[] => {
    return response.products.map(product => ({
      ...product,
      brand: product.brand || 'Без бренду'
    }))
  },
  lazy: true
})

const columns: TableColumn<Product>[] = [
  {
    accessorKey: 'title',
    header: 'Назва',
    enableSorting: true
  },
  {
    accessorKey: 'description',
    header: 'Опис',
    enableSorting: false,
    cell: ({ row }) => h('div', {
      style: 'max-width: 500px; white-space: normal; word-break: break-word;'
    }, row.original.description)
  },
  {
    accessorKey: 'price',
    header: 'Ціна, $',
    enableSorting: true
  },
  {
    accessorKey: 'rating',
    header: 'Рейтинг',
    enableSorting: true,
    cell: ({ row }) => {
      const rating = row.original.rating
      const color = rating < 4.5 ? 'text-red-600' : 'text-green-600'
      return h('span', { class: color }, rating.toFixed(1))
    }
  },
  {
    accessorKey: 'brand',
    header: 'Бренд',
    enableSorting: true
  },
  {
    accessorKey: 'category',
    header: 'Категорія',
    enableSorting: true
  },
  {
    accessorKey: 'thumbnail',
    header: 'Фото',
    enableSorting: false,
    cell: ({ row }) => {
      const url = row.original.thumbnail
      return h('img', {
        src: url,
        width: 100,
        height: 100,
        style: 'object-fit: cover; border-radius: 12px; border: 1px solid #ccc'
      })
    }
  }
]

const globalFilter = ref('')
const pagination = ref({ pageIndex: 0, pageSize: 6 })
const sorting = ref([])
const handleFilterChange = () => pagination.value.pageIndex = 0
watch(globalFilter, handleFilterChange)
</script>

<template>
  <div class="w-full px-10 mx-auto space-y-4 pb-4">
    <div class="flex px-4 py-4 border-b border-gray-200">
      <UInput v-model="globalFilter" class="max-w-md" placeholder="Пошук товарів..." />
    </div>

    <UTable
        ref="table"
        v-model:global-filter="globalFilter"
        v-model:pagination="pagination"
        v-model:sorting="sorting"
        :data="data || []"
        :columns="columns"
        :pagination-options="{ getPaginationRowModel: getPaginationRowModel() }"
        :sort-options="{ getSortedRowModel: getSortedRowModel() }"
        class="w-full text-left border border-gray-300 divide-y divide-gray-200"
    />

    <div class="flex justify-center border-t border-gray-200 pt-4">
      <UPagination
          :default-page="(table?.tableApi?.getState().pagination.pageIndex || 0) + 1"
          :items-per-page="table?.tableApi?.getState().pagination.pageSize"
          :total="table?.tableApi?.getFilteredRowModel().rows.length"
          @update:page="(p) => table?.tableApi?.setPageIndex(p - 1)"
      />
    </div>
  </div>
</template>
