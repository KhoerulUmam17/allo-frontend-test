<template>
  <v-container>
    <div v-if="loading">
      <v-progress-circular indeterminate color="primary" />
      <span>Memuat detail roket...</span>
    </div>
    <div v-else-if="error">
      <v-alert type="error">
        Gagal memuat detail roket.
        <v-btn color="primary" @click="fetchRocket">Coba Lagi</v-btn>
      </v-alert>
    </div>
    <div v-else-if="rocket">
      <v-row>
        <v-col cols="12" md="6">
          <v-img :src="rocket.image" height="300" />
        </v-col>
        <v-col cols="12" md="6">
          <h2>{{ rocket.name }}</h2>
          <p>{{ rocket.description }}</p>
          <v-list>
            <v-list-item>
              <v-list-item-title>Biaya per peluncuran</v-list-item-title>
              <v-list-item-subtitle>{{ formatCurrency(rocket.cost_per_launch) }}</v-list-item-subtitle>
            </v-list-item>
            <v-list-item>
              <v-list-item-title>Negara</v-list-item-title>
              <v-list-item-subtitle>{{ rocket.country }}</v-list-item-subtitle>
            </v-list-item>
            <v-list-item>
              <v-list-item-title>Penerbangan pertama</v-list-item-title>
              <v-list-item-subtitle>{{ rocket.first_flight }}</v-list-item-subtitle>
            </v-list-item>
          </v-list>
        </v-col>
      </v-row>
    </div>
  </v-container>
</template>

<script lang="ts" setup>
import { ref, onMounted } from 'vue';
import { useRoute } from 'vue-router';

const route = useRoute();
const rocket = ref<any>(null);
const loading = ref(false);
const error = ref(false);

const fetchRocket = async () => {
  loading.value = true;
  error.value = false;
  try {
    const id = route.params.id;
    const res = await fetch(`https://api.spacexdata.com/v4/rockets/${id}`);
    const data = await res.json();
    rocket.value = {
      id: data.id,
      name: data.name,
      description: data.description,
      image: data.flickr_images[0] || '',
      cost_per_launch: data.cost_per_launch,
      country: data.country,
      first_flight: data.first_flight
    };
  } catch (e) {
    error.value = true;
  } finally {
    loading.value = false;
  }
};

const formatCurrency = (val: number) => {
  return val ? `$${val.toLocaleString()}` : '-';
};

onMounted(fetchRocket);
</script>
