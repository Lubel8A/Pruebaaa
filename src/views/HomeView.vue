<template>
  <div class="container">
    <h1 class="title">Subir y Analizar Archivo Excel</h1>
    <ExcelUpload />
  </div>
</template>

<script>
import ExcelUpload from '@/components/ExcelUpload.vue';

export default {
  name: 'App',
  components: {
    ExcelUpload,
  },
};
</script>

<style>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

.title {
  font-size: 2.5rem;
  margin-bottom: 20px;
}
</style>

  