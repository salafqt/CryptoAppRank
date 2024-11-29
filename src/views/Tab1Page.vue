<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Crypto Prices</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content>
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Crypto Prices</ion-title>
        </ion-toolbar>
      </ion-header>
      <div
        style="
          display: flex;
          flex-direction: column;
          align-items: center;
          margin-top: 20px;
        "
      >
        <ion-button @click="fetchData">Refresh Data</ion-button>
        <ion-input
          v-model="searchQuery"
          placeholder="Search by name"
          class="search-input"
        ></ion-input>
      </div>
      <ion-grid>
        <ion-row>
          <ion-col>Rank</ion-col>
          <ion-col>Symbol & Name</ion-col>
          <ion-col>USD & Price</ion-col>
        </ion-row>
        <ion-row v-for="coin in paginatedFilteredCoins" :key="coin.id">
          <ion-col>{{ coin.rank }}</ion-col>
          <ion-col>
            <div>{{ coin.symbol }}</div>
            <div>{{ coin.name }}</div>
          </ion-col>
          <ion-col>
            <div>USD</div>
            <div>{{ coin.price_usd }}</div>
          </ion-col>
        </ion-row>
      </ion-grid>
      <div style="display: flex; justify-content: center; margin-top: 20px">
        <ion-button @click="prevPage" :disabled="currentPage === 1"
          >Previous</ion-button
        >
        <ion-button @click="nextPage" :disabled="currentPage === totalPages"
          >Next</ion-button
        >
      </div>
    </ion-content>

    <ion-alert
      :is-open="showAlert"
      :header="alertHeader"
      :message="alertMessage"
      :buttons="['OK']"
      @didDismiss="showAlert = false"
    ></ion-alert>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonButton,
  IonGrid,
  IonRow,
  IonCol,
  IonAlert,
  IonInput,
} from "@ionic/vue";
import { ref, computed, onMounted } from "vue";
import axios from "axios";

// Define an interface for the coin objects
interface Coin {
  id: number;
  rank: number;
  name: string;
  symbol: string;
  price_usd: string;
}

const coins = ref<Coin[]>([]);
const currentPage = ref(1);
const itemsPerPage = 10;
const showAlert = ref(false);
const alertHeader = ref("");
const alertMessage = ref("");
const searchQuery = ref("");

const fetchData = async () => {
  try {
    const response = await axios.get("https://api.coinlore.net/api/tickers/");
    coins.value = response.data.data;
    alertHeader.value = "Success";
    alertMessage.value =
      "The cryptocurrency data has been refreshed successfully.";
  } catch (error) {
    console.error("Error fetching data:", error);
    alertHeader.value = "Error";
    alertMessage.value = "Failed to refresh data. Please try again.";
  } finally {
    showAlert.value = true;
  }
};

// Fetch data when the component is mounted
onMounted(() => {
  fetchData();
});

const filteredCoins = computed(() => {
  return coins.value.filter((coin) =>
    coin.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});

const paginatedFilteredCoins = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return filteredCoins.value.slice(start, end);
});

const totalPages = computed(() => {
  return Math.ceil(filteredCoins.value.length / itemsPerPage);
});

const nextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value++;
  }
};

const prevPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--;
  }
};
</script>

<style scoped>
ion-grid {
  margin-top: 20px;
}
ion-row {
  border-bottom: 1px solid #ccc;
  padding: 10px 0;
}
ion-col {
  text-align: center;
}
.search-input {
  margin-top: 10px;
  border: 1px solid #ccc;
  padding: 10px;
  width: 80%;
}
</style>
