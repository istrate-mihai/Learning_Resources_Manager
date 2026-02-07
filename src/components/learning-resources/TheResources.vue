<script setup>
import { ref, computed, provide, onMounted } from 'vue';
import AddResource from './AddResource.vue';
import StoredResources from './StoredResources.vue';

const selectedTab     = ref('stored-resources');
const isLoading       = ref(false);
const errors          = ref(false);
const storedResources = ref([]);

async function fetchData() {
  isLoading.value = true;

  try {
    const response = await fetch('https://vue-http-demo-87aee-default-rtdb.firebaseio.com/resources.json');
    if (response.ok) {
      const results   = [];
      const resources = await response.json();

      if (resources !== null) {
        for (let key in resources) {
          results.push(resources[key]);
          results[results.length - 1]['key'] = key;
          storedResources.value              = results;
        }
      }
    }
  }
  catch (error) {
    console.error(error);
    errors.value = true;
  }
  finally {
    isLoading.value = false;
  }
}

function setSelectedTab(tab) {
  selectedTab.value = tab;
}

async function addResource(title, description, url) {
  isLoading.value = true;

  const newResource = {
    id: new Date().toISOString(),
    title: title,
    description: description,
    link: url
  };

   try {
    const response = await fetch('https://vue-http-demo-87aee-default-rtdb.firebaseio.com/resources.json', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(newResource),
    });

    if (!response.ok) {
      throw new Error('Could not save data!');
    }

    await fetchData();
    selectedTab.value = 'stored-resources';
  } catch (error) {
    console.error(error);
    errors.value = error.message;
  } finally {
    isLoading.value = false;
  }
}

async function removeResource(resourceKey) {
  isLoading.value = true;

  try {
    const response = await fetch(
      `https://vue-http-demo-87aee-default-rtdb.firebaseio.com/resources/${resourceKey}.json`,
      {
        method: 'DELETE',
      }
    );

    if (!response.ok) {
      throw new Error(`Failed to delete resource. Status: ${response.status}`);
    }
    storedResources.value = [];

    await fetchData();

    return true;
  } catch (error) {
    console.error('Delete error:', error);
    throw error;
  }
  finally {
    isLoading.value = false;
  }
}

const storedResButtonMode = computed(() => {
  return selectedTab.value == 'stored-resources' ? null : 'flat';
});

const addResButtonMode = computed(() => {
  return selectedTab.value == 'add-resource' ? null : 'flat'
});

provide('resources', storedResources);
provide('addResource', addResource);
provide('removeResource', removeResource);
provide('isLoading', isLoading);

onMounted(() => {
  fetchData();
});
</script>

<template>
  <base-card>
    <base-button
      @click="setSelectedTab('stored-resources')"
      :mode="storedResButtonMode"
    >Stored Resources</base-button>

    <base-button
      @click="setSelectedTab('add-resource')"
      :mode="addResButtonMode"
    >Add Resource</base-button>
  </base-card>

  <keep-alive>
    <component :is="selectedTab === 'stored-resources' ? StoredResources : AddResource"></component>
  </keep-alive>
</template>
