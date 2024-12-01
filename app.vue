<script lang="ts" setup>
import { ref, watch, computed } from 'vue';
import type { IProductResponse, IProduct } from '../list-of-products/index.vue';

export interface ISortable {
  column: string;
  direction: string;
}

const sort = ref<ISortable>();
const q = ref('');
const page = ref(1);
const pageCount = 10;

const columns = [
  { key: 'price', label: 'Ціна', sortable: true },
  { key: 'title', label: 'Назва' },
  { key: 'description', label: 'Опис' },
  { key: 'rating', label: 'Оцінка', sortable: true },
  { key: 'brand', label: 'Бренд' },
  { key: 'category', label: 'Категорія' },
  { key: 'thumbnail', label: 'Фото' },
];

const { data, pending } = await useLazyAsyncData<IProductResponse>(
    'products',
    () => $fetch(`https://dummyjson.com/products?limit=100`)
);

const products = ref<IProduct[]>(data.value?.products || []);
const total = ref(data.value?.total || 0);

watch(pending, () => {
  if (data.value) {
    products.value = data.value.products;
    total.value = data.value.total;
  }
});

const filteredRows = computed(() => {
  if (!q.value) return products.value;
  page.value = 1;
  return products.value.filter((p: IProduct) =>
      Object.values(p).some(value =>
          String(value).toLowerCase().includes(q.value.toLowerCase())
      )
  );
});

const sortMethod = (a: IProduct, b: IProduct) => {
  if (!sort.value?.column) return 0;
  const key = sort.value.column as keyof IProduct;
  const dir = sort.value.direction === 'desc' ? -1 : 1;
  return dir * (a[key] > b[key] ? 1 : a[key] < b[key] ? -1 : 0);
};

const rows = computed(() => {
  const start = (page.value - 1) * pageCount;
  const end = page.value * pageCount;
  return filteredRows.value.sort(sortMethod).slice(start, end);
});
</script>

<template>
  <title>Список продуктів</title>
  <UCard class="w-full" :ui="{ header: { padding: 'px-4 py-5' } }">
    <div class="flex px-3 py-3.5 border-b">
      <UInput v-model="q" placeholder="Filter..." />
    </div>

    <UTable
        v-model:sort="sort"
        :rows="rows"
        :columns="columns"
        :loading="pending"
        sort-asc-icon="i-heroicons-arrow-up"
        sort-desc-icon="i-heroicons-arrow-down"
        :ui="{ td: { base: 'max-w-[0] truncate' } }"
    >
      <template #thumbnail-data="{ row }">
        <img class="w-[100px] h-[100px]" :src="row.thumbnail" alt="Thumbnail" />
      </template>
      <template #rating-data="{ row }">
                <span :class="row.rating < 4.5 ? 'text-red-700' : 'text-green-700'">
                    {{ row.rating }}
                </span>
      </template>
    </UTable>

    <div class="flex justify-end px-3 py-3.5 border-t">
      <UPagination v-model="page" :page-count="pageCount" :total="total" />
    </div>
  </UCard>
</template>
