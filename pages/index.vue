<template>
  <div class="container mx-auto p-4">
    <h1 class="text-4xl font-bold text-center text-gray-900 dark:text-white mb-10">
      Nuestros Productos Destacados
    </h1>

    <div v-if="pending" class="flex justify-center items-center h-64">
      <UProgress animation="carousel" class="w-1/2" />
      <p class="ml-4 text-gray-600 dark:text-gray-300">Cargando productos...</p>
    </div>

    <div v-else-if="error">
      <UAlert
        icon="i-heroicons-exclamation-triangle"
        color="red"
        variant="soft"
        title="Error al cargar productos"
        :description="`No pudimos cargar los productos: ${error.message}`"
        class="mt-8"
      />
    </div>

    <div v-else-if="products && products.length > 0" class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
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
        description="Parece que no se encontraron productos en este momento. Por favor, inténtalo de nuevo más tarde."
        class="mt-8"
      />
    </div>
  </div>
</template>

<script setup>
// Usamos `useAsyncData` para cargar el JSON de forma asíncrona en el servidor (SSR)
// y también en el cliente si es necesario. Esto es ideal para archivos locales.
const { data: products, pending, error } = await useAsyncData('all-products', () =>
  // Importamos el archivo JSON directamente. Nuxt lo manejará como un módulo.
  import('~/data/products.json').then(m => m.default)
);
</script>