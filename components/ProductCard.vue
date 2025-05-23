<template>
  <UCard
    class="product-card cursor-pointer hover:shadow-lg transition-shadow duration-200 ease-in-out"
    @click="navigateToProductPage"
  >
    <div class="relative w-full h-48 mb-4">
      <NuxtImg
        :src="product.photos[0]"
        :alt="product.name"
        class="w-full h-full object-contain rounded-md"
        loading="lazy"
        provider="ipx"
      />
      </div>

    <h3 class="text-lg font-semibold text-gray-900 dark:text-white mb-1">
      {{ product.name }}
    </h3>

    <p class="text-sm text-gray-600 dark:text-gray-300 mb-3 line-clamp-2">
      {{ product.shortDescription }}
    </p>

    <div class="flex items-center justify-between">
      <p class="text-xl font-bold text-primary-600 dark:text-primary-400">
        {{ formattedPrice }}
      </p>
      <UButton
        icon="i-heroicons-arrow-right"
        size="sm"
        color="primary"
        variant="solid"
        label="Ver más"
        :to="`/products/${product.id}`"
        @click.stop="navigateToProductPage"
      />
    </div>
  </UCard>
</template>

<script setup>
import { defineProps, computed } from 'vue';
import { useRouter } from 'vue-router';

const props = defineProps({
  product: {
    type: Object,
    required: true,
    // Define la forma esperada del objeto product
    default: () => ({
      id: '',
      name: 'Nombre del Producto',
      shortDescription: 'Descripción corta del producto. Aquí se puede ver un breve resumen de sus características principales.',
      imageUrl: 'https://via.placeholder.com/400x300/F3F4F6/9CA3AF?text=Producto', // Placeholder por defecto
      price: 0,
    }),
  },
});

const router = useRouter();

const formattedPrice = computed(() => {
  return new Intl.NumberFormat('es-MX', {
    style: 'currency',
    currency: 'MXN', // Moneda mexicana, cámbiala si es necesario
  }).format(props.product.variants.weight[0].price);
});

const navigateToProductPage = () => {
  router.push(`/products/${props.product.id}`);
};
</script>

<style scoped>
/* Puedes añadir estilos adicionales aquí si lo necesitas */
.product-card {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  height: 100%; /* Asegura que todas las tarjetas tengan la misma altura si están en una cuadrícula */
}
</style>