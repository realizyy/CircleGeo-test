<template>
    <div class="layer-controls bg-[#00000059] rounded-lg fixed bottom-0 left-0 right-0 p-2 z-10">
      <div class="mobile-toggle" @click="isExpanded = !isExpanded" v-if="isMobile">
        <button class="btn bg-gray-600 text-white py-2 px-4 rounded w-full">
          {{ isExpanded ? 'Sembunyikan Kontrol' : 'Tampilkan Kontrol' }}
        </button>
      </div>
      <div class="button-container flex flex-wrap justify-center items-center" :class="{ 'hidden': isMobile && !isExpanded }">
        <button
          v-for="layer in layers"
          :key="layer"
          class="btn bg-blue-500 text-white py-2 px-4 rounded m-1"
          @click="toggleLayer(layer)"
        >
          Toggle {{ capitalizeFirstLetter(layer) }}
        </button>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        layers: ['line', 'polygon', 'circle', 'customMarker', 'defaultMarker'],
        isExpanded: false,
        isMobile: false,
      };
    },
    mounted() {
      this.checkIsMobile();
      window.addEventListener('resize', this.checkIsMobile);
    },
    beforeUnmounted() {
      window.removeEventListener('resize', this.checkIsMobile);
    },
    methods: {
      toggleLayer(layer) {
        this.$emit('toggle-layer', layer);
      },
      capitalizeFirstLetter(string) {
          return string.charAt(0).toUpperCase() + string.slice(1);
      },
      checkIsMobile() {
        this.isMobile = window.innerWidth <= 768;
      },
    }
  };
  </script>
  
  <style scoped>
  .layer-controls {
    margin: 1rem;
  }
  
  .button-container {
      gap: 0.5rem;
      transition: height 0.3s ease-in-out, opacity 0.3s ease-in-out;
  }
  
  .mobile-toggle {
      margin-bottom: 0.5rem;
  }
  
  @media (max-width: 768px) {
    .layer-controls {
      padding: 0.5rem;
      margin: 0.25rem;
    }
    .button-container {
        flex-direction: column;
        align-items: stretch;
        overflow: hidden;
    }
    .button-container .btn {
        margin: 0.25rem 0;
        width: 100%;
    }
  }
  </style>