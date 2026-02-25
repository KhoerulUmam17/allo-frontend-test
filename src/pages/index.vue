
<script lang="ts" setup>
import { ref, computed, onMounted } from 'vue';
import { useRouter } from 'vue-router';

const rockets = ref<any[]>([]);
const loading = ref(false);
const error = ref(false);
const filter = ref('');
const showAddRocket = ref(false);
const newRocket = ref({ name: '', description: '', image: '' });
const router = useRouter();

const fetchRockets = async () => {
  loading.value = true;
  error.value = false;
  try {
    const res = await fetch('https://api.spacexdata.com/v4/rockets');
    const data = await res.json();
    rockets.value = data.map((r: any) => ({
      id: r.id,
      name: r.name,
      description: r.description,
      image: r.flickr_images[0] || '',
      ...r
    }));
  } catch (e) {
    error.value = true;
  } finally {
    loading.value = false;
  }
};

const filteredRockets = computed(() => {
  if (!filter.value) return rockets.value;
  return rockets.value.filter(r =>
    r.name.toLowerCase().includes(filter.value.toLowerCase()) ||
    r.description.toLowerCase().includes(filter.value.toLowerCase())
  );
});

const addRocket = () => {
  rockets.value.unshift({
    id: Date.now(),
    name: newRocket.value.name,
    description: newRocket.value.description,
    image: newRocket.value.image
  });
  showAddRocket.value = false;
  newRocket.value = { name: '', description: '', image: '' };
};

const goToDetail = (rocket: any) => {
  router.push({ name: 'rocket-detail', params: { id: rocket.id } });
};

onMounted(fetchRockets);
</script>

<template>
  <v-container>
    <v-row>
      <v-col cols="12" md="8">
        <v-text-field
          v-model="filter"
          label="Filter roket"
          prepend-inner-icon="mdi-filter"
          class="mb-4"
          @input="() => {}"
          clearable
        />
      </v-col>
      <v-col cols="12" md="4">
        <v-btn color="primary" @click="showAddRocket = true">Tambah Roket</v-btn>
      </v-col>
    </v-row>

    <v-dialog v-model="showAddRocket" max-width="500">
      <v-card>
        <v-card-title>Tambah Roket Baru</v-card-title>
        <v-card-text>
          <v-text-field v-model="newRocket.name" label="Nama Roket" />
          <v-text-field v-model="newRocket.description" label="Deskripsi" />
          <v-text-field v-model="newRocket.image" label="URL Gambar" />
        </v-card-text>
        <v-card-actions>
          <v-btn color="primary" @click="addRocket">Tambah</v-btn>
          <v-btn text @click="showAddRocket = false">Batal</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <div v-if="loading">
      <v-progress-circular indeterminate color="primary" />
      <span>Memuat data roket...</span>
    </div>
    <div v-else-if="error">
      <v-alert type="error">
        Gagal memuat data roket.
        <v-btn color="primary" @click="fetchRockets">Coba Lagi</v-btn>
      </v-alert>
    </div>
    <div v-else>
      <v-row>
        <v-col v-for="rocket in filteredRockets" :key="rocket.id" cols="12" md="6" lg="4">
          <v-card @click="goToDetail(rocket)" class="mb-4" hover>
            <v-img :src="rocket.image" height="200" />
            <v-card-title>{{ rocket.name }}</v-card-title>
            <v-card-text>{{ rocket.description }}</v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </div>

  </v-container>
</template>
