<template>
  <div class="container mx-auto p-8">
    <UButton
      icon="heroicons:arrow-left"
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
      <div class="relative w-full h-96 md:h-[400px] lg:h-[500px] flex items-center justify-center bg-gray-100 dark:bg-gray-800 rounded-lg shadow-md overflow-hidden">
        <NuxtImg
          v-if="product.photos && product.photos.length > 0"
          :src="product.photos[currentImageIndex]"
          :alt="`${product.name} - Imagen ${currentImageIndex + 1}`"
          class="w-full h-full object-contain transition-opacity duration-300 ease-in-out"
          provider="ipx"
          loading="eager"
        />
        <div v-else class="text-gray-500 dark:text-gray-400">
          <UIcon name="heroicons:photo" class="w-12 h-12 mb-2" />
          <p>No hay imágenes disponibles</p>
        </div>

        <UButton
          v-if="product.photos && product.photos.length > 1"
          icon="heroicons:chevron-left"
          variant="ghost"
          color="gray"
          size="lg"
          class="absolute left-2 top-1/2 -translate-y-1/2 z-10 p-2 bg-black/30 hover:bg-black/50 rounded-full"
          aria-label="Imagen anterior"
          @click.stop="prevImage"
        />

        <UButton
          v-if="product.photos && product.photos.length > 1"
          icon="heroicons:chevron-right"
          variant="ghost"
          color="gray"
          size="lg"
          class="absolute right-2 top-1/2 -translate-y-1/2 z-10 p-2 bg-black/30 hover:bg-black/50 rounded-full"
          aria-label="Imagen siguiente"
          @click.stop="nextImage"
        />

        <div v-if="product.photos && product.photos.length > 1" class="absolute bottom-4 flex space-x-2 z-10">
          <span
            v-for="(photo, index) in product.photos"
            :key="index"
            class="block w-2 h-2 rounded-full cursor-pointer transition-all duration-200"
            :class="{'bg-primary-500 scale-125': index === currentImageIndex, 'bg-gray-400 dark:bg-gray-600': index !== currentImageIndex}"
            @click="setCurrentImage(index)"
            :aria-label="`Ir a imagen ${index + 1}`"
          ></span>
        </div>
      </div>

      <div>
        <h1 class="text-4xl lg:text-5xl font-extrabold text-gray-900 dark:text-white mb-4">
          {{ product.name }}
        </h1>
        <p class="text-lg text-gray-700 dark:text-gray-300 mb-6 leading-relaxed">
          {{ product.longDescription }} </p>

        <div v-if="product.brand" class="mb-6 p-4 bg-gray-50 dark:bg-gray-800 rounded-lg">
          <h3 class="text-xl font-semibold text-gray-900 dark:text-white mb-2">{{ product.brand.name }}</h3>
          <p class="text-sm text-gray-600 dark:text-gray-400">{{ product.brand.description }}</p>
        </div>

        <p class="text-3xl lg:text-4xl font-extrabold text-primary-700 dark:text-primary-300 mb-6">
          {{ formattedPrice }}
        </p>

        <div v-if="product.variants?.weight?.length" class="mb-6">
          <label for="weight-select" class="block text-sm font-medium text-gray-700 dark:text-gray-200 mb-2">
            Selecciona el peso:
          </label>
          <USelect
            id="weight-select"
            v-model="selectedWeight"
            :items="weightOptions"
            option-attribute="label"
            value-attribute="value"
            placeholder="Selecciona una opción"
            class="max-w-xs"
          />
        </div>

        <div v-if="product.variants?.grind?.length" class="mb-6">
          <label for="grind-select" class="block text-sm font-medium text-gray-700 dark:text-gray-200 mb-2">
            Selecciona el tipo de molido:
          </label>
          <USelect
            id="grind-select"
            v-model="selectedGrind"
            :items="product.variants.grind"
            placeholder="Selecciona una opción"
            class="max-w-xs"
          />
        </div>


        <UButton
          icon="heroicons:shopping-cart"
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
        icon="heroicons:exclamation-triangle"
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
import { ref, computed, watch } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const router = useRouter();

const currentImageIndex = ref(0);
const selectedWeight = ref(null); // Para almacenar la opción de peso seleccionada
const selectedGrind = ref(null); // Para almacenar la opción de molido seleccionada

// Carga de datos del producto
const { data: productsData, pending, error } = await useAsyncData('products-detail', () =>
  import('~/data/products.json').then(m => m.default)
);

// Propiedad computada para obtener el producto actual
const product = computed(() => {
  if (productsData.value) {
    const foundProduct = productsData.value.find(p => p.id === route.params.id);
    // Inicializa las selecciones de variante cuando el producto se carga por primera vez
    if (foundProduct && foundProduct.variants?.weight?.length && !selectedWeight.value) {
      selectedWeight.value = foundProduct.variants.weight[0].value;
    }
    if (foundProduct && foundProduct.variants?.grind?.length && !selectedGrind.value) {
      selectedGrind.value = foundProduct.variants.grind[0];
    }
    return foundProduct;
  }
  return null;
});

// Watcher para resetear el índice de la imagen y selecciones de variantes cuando el producto cambia
watch(product, (newProduct) => {
  currentImageIndex.value = 0;
  if (newProduct) {
    if (newProduct.variants?.weight?.length) {
      selectedWeight.value = newProduct.variants.weight[0].value;
    } else {
      selectedWeight.value = null;
    }
    if (newProduct.variants?.grind?.length) {
      selectedGrind.value = newProduct.variants.grind[0];
    } else {
      selectedGrind.value = null;
    }
  }
}, { immediate: true }); // 'immediate' para que se ejecute la primera vez que 'product' tenga un valor

// Opciones de peso para el USelect
const weightOptions = computed(() => {
  console.log("w1");
  if (product.value?.variants?.weight) {
    console.log("w2");
    return product.value.variants.weight.map(w => ({
      label: `${w.value} - ${new Intl.NumberFormat('es-MX', { style: 'currency', currency: 'MXN' }).format(w.discountPrice || w.price)}`,
      value: w.value
    }));
  }
  console.log("w3");
  return [];
});

// Precio formateado (ahora considera la variante seleccionada)
const formattedPrice = computed(() => {
  if (!product.value || !selectedWeight.value) return '';

  const selectedVariant = product.value.variants?.weight?.find(
    (w) => w.value === selectedWeight.value
  );

  if (selectedVariant) {
    const displayPrice = selectedVariant.discountPrice !== undefined && selectedVariant.discountPrice < selectedVariant.price
      ? selectedVariant.discountPrice
      : selectedVariant.price;
    return new Intl.NumberFormat('es-MX', {
      style: 'currency',
      currency: 'MXN',
    }).format(displayPrice);
  }
  return '';
});


// Funciones para el carrusel
const nextImage = () => {
  if (product.value && product.value.photos) {
    currentImageIndex.value = (currentImageIndex.value + 1) % product.value.photos.length;
  }
};

const prevImage = () => {
  if (product.value && product.value.photos) {
    currentImageIndex.value = (currentImageIndex.value - 1 + product.value.photos.length) % product.value.photos.length;
  }
};

const setCurrentImage = (index) => {
  if (product.value && product.value.photos && index >= 0 && index < product.value.photos.length) {
    currentImageIndex.value = index;
  }
};

// Función de agregar al carrito (ahora con variantes seleccionadas)
const addToCart = (productToAdd) => {
  console.log('Producto agregado al carrito:', {
    id: productToAdd.id,
    name: productToAdd.name,
    selectedWeight: selectedWeight.value,
    selectedGrind: selectedGrind.value,
    price: formattedPrice.value // O el precio exacto de la variante seleccionada
  });
  // Aquí integrarías tu lógica real para agregar al carrito,
  // pasando el ID del producto y las variantes seleccionadas.
};
</script>

<style scoped>
/* Puedes añadir estilos adicionales aquí si lo necesitas */
</style>