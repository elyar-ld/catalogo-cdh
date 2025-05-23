<template>
  <div>
    <UContainer>
      <NuxtPage />
    </UContainer>
  </div>
</template>

<!--template>
  <div class="container mx-auto p-4">
    <h2 class="text-3xl font-bold text-gray-900 dark:text-white mb-8">Nuestros Productos</h2>
    <div v-if="products && products.length" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
      <ProductCard
        v-for="product in products"
        :key="product.id"
        :product="product"
      />
    </div>
    <div v-else>
      <UAlert
        icon="i-heroicons-information-circle"
        color="blue"
        variant="soft"
        title="No hay productos disponibles"
        description="Parece que no se encontraron productos en este momento."
        class="mt-8"
      />
    </div>
  </div>
</template>

<script setup>
// Usamos `useAsyncData` para cargar el JSON de forma asíncrona
const { data: products, pending, error } = await useAsyncData('all-products', () =>
  import('~/data/products.json').then(m => m.default)
);

// Puedes manejar `pending` y `error` en tu plantilla si quieres mostrar
// un loader o un mensaje de error global
</script-->

<style>
  @import "tailwindcss";
  @import "@nuxt/ui";
</style>