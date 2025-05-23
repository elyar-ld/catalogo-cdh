<template>
  <div class="container mx-auto p-8">
    <UButton
      icon="i-heroicons-arrow-left"
      variant="ghost"
      label="Volver a los productos"
      class="mb-6"
      @click="router.back()"
    />

    <div v-if="pending" class="flex justify-center items-center h-96">
      <UProgress animation="carousel" class="w-1/2" />
      <p class="ml-4 text-gray-600 dark:text-gray-300">Cargando detalles del producto...</p>
    </div>

    <div v-else-if="product" class="grid grid-cols-1 md:grid-cols-2 gap-8 lg:gap-12 items-start">
      <div class="relative w-full h-96 md:h-[400px] lg:h-[500px]">
        <NuxtImg
          :src="product.imageUrl"
          :alt="product.name"
          class="w-full h-full object-contain rounded-lg shadow-md transition-transform duration-300 hover:scale-[1.02]"
          provider="ipx"
          loading="eager"
        />
      </div>
      <div>
        <h1 class="text-4xl lg:text-5xl font-extrabold text-gray-900 dark:text-white mb-4">
          {{ product.name }}
        </h1>
        <p class="text-lg text-gray-700 dark:text-gray-300 mb-6 leading-relaxed">
          {{ product.fullDescription }}
        </p>
        <p class="text-3xl lg:text-4xl font-extrabold text-primary-700 dark:text-primary-300 mb-6">
          {{ formattedPrice }}
        </p>
        <UButton
          icon="i-heroicons-shopping-cart"
          size="lg"
          color="primary"
          variant="solid"
          label="Agregar al Carrito"
          block
          @click="addToCart(product)"
          class="mt-4"
        />
        </div>
    </div>

    <div v-else>
      <UAlert
        icon="i-heroicons-exclamation-triangle"
        color="red"
        variant="soft"
        title="Producto no encontrado"
        description="Lo sentimos, el producto que buscas no existe o no se pudo cargar correctamente. Por favor, verifica el ID."
        class="mt-8"
      />
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const router = useRouter();
const product = ref(null); // Aquí se almacenarán los datos del producto

// Usamos `useAsyncData` para cargar el JSON de forma asíncrona
// Esto es ideal para archivos locales y garantiza que los datos estén disponibles para SSR
const { data: productsData, pending, error } = await useAsyncData('products-detail', () =>
  import('~/data/products.json').then(m => m.default)
);

// Buscamos el producto específico una vez que `productsData` esté disponible
// y el ID de la ruta haya sido resuelto.
product.value = computed(() => {
  if (productsData.value) {
    return productsData.value.find(p => p.id === route.params.id);
  }
  return null;
});

const formattedPrice = computed(() => {
  if (!product.value) return '';
  return new Intl.NumberFormat('es-MX', {
    style: 'currency',
    currency: 'MXN', // Moneda mexicana, cámbiala si es necesario
  }).format(product.value.price);
});

const addToCart = (productToAdd) => {
  // Aquí iría tu lógica para agregar el producto al carrito.
  // Podrías usar un store de Pinia, un composable, o emitir un evento.
  console.log('Producto agregado al carrito:', productToAdd.name);

  // Ejemplo: mostrar una notificación con nuxt/ui
  // const toast = useToast(); // Descomenta y asegúrate de tener `useToast` disponible
  // toast.add({
  //   title: '¡Producto añadido!',
  //   description: `${productToAdd.name} ha sido agregado a tu carrito.`,
  //   icon: 'i-heroicons-check-circle',
  //   color: 'green'
  // });
};
</script>

<style scoped>
/* Puedes añadir estilos adicionales aquí si lo necesitas */
</style>