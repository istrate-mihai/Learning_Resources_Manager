<script setup>
import LearningResource from './LearningResource.vue';
import { inject } from 'vue';

const resources = inject('resources');
const isLoading = inject('isLoading');
</script>

<template>
  <base-card v-if="isLoading">
    <p style="text-align: center;">Loading...</p>
  </base-card>
  <ul v-else-if="!isLoading && resources.length !== 0">
    <learning-resource
      v-for="(res, index) in resources"
      :key="res.id"
      :id="index.toString()"
      :resourceKey="res.key"
      :title="res.title"
      :description="res.description"
      :link="res.link"
    >
    </learning-resource>
  </ul>
  <base-card v-else>
    <p style="text-align: center;">No resources stored at this time.</p>
  </base-card>

</template>

<style scoped>
ul {
  list-style: none;
  margin: 0;
  padding: 0;
  margin: auto;
  max-width: 40rem;
}
</style>
