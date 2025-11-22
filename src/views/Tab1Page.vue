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

      <div class="controls">
        <div class="top-row">
          <ion-button size="small" color="primary" @click="fetchData"
            >Refresh</ion-button
          >
          <ion-input
            v-model="searchQuery"
            placeholder="Search by name or symbol"
            class="search-input"
            clear-input
          ></ion-input>
        </div>
      </div>

      <ion-grid class="crypto-table">
        <ion-row class="table-head">
          <ion-col>Rank</ion-col>
          <ion-col>Symbol & Name</ion-col>
          <ion-col>USD & Price</ion-col>
        </ion-row>

        <ion-row
          v-for="coin in paginatedFilteredCoins"
          :key="coin.id"
          class="table-row"
        >
          <ion-col class="rank-col">{{ coin.rank }}</ion-col>
          <ion-col class="name-col">
            <div class="symbol">{{ coin.symbol }}</div>
            <div class="fullname">{{ coin.name }}</div>
          </ion-col>
          <ion-col class="price-col">
            <div class="currency">USD</div>
            <div class="amount">{{ formatPrice(coin.price_usd) }}</div>
          </ion-col>
        </ion-row>

        <ion-row v-if="filteredCoins.length === 0">
          <ion-col class="no-results">No coins found.</ion-col>
        </ion-row>
      </ion-grid>

      <div class="pagination-controls">
        <ion-button
          size="small"
          fill="outline"
          @click="prevPage"
          :disabled="currentPage === 1"
          >Prev</ion-button
        >

        <ion-button
          v-for="p in visiblePages"
          :key="String(p) + '-pg'"
          size="small"
          :fill="currentPage === p ? 'solid' : 'outline'"
          :disabled="p === '...'"
          @click="typeof p === 'number' && goToPage(p as number)"
        >
          {{ p }}
        </ion-button>

        <ion-button
          size="small"
          fill="outline"
          @click="nextPage"
          :disabled="currentPage === totalPages"
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
import { ref, computed, onMounted, watch } from "vue";
import axios from "axios";

interface Coin {
  id: string;
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
    alertMessage.value = "Cryptocurrency data refreshed.";
  } catch (error) {
    console.error("Error fetching data:", error);
    alertHeader.value = "Error";
    alertMessage.value = "Failed to refresh data. Please try again.";
  } finally {
    showAlert.value = true;
  }
};

onMounted(() => {
  fetchData();
});

watch(searchQuery, () => {
  currentPage.value = 1;
});

const filteredCoins = computed(() => {
  const q = searchQuery.value.trim().toLowerCase();
  if (!q) return coins.value;
  return coins.value.filter(
    (coin) =>
      coin.name.toLowerCase().includes(q) ||
      coin.symbol.toLowerCase().includes(q)
  );
});

const paginatedFilteredCoins = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage;
  const end = start + itemsPerPage;
  return filteredCoins.value.slice(start, end);
});

const totalPages = computed(() => {
  return Math.max(1, Math.ceil(filteredCoins.value.length / itemsPerPage));
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

const goToPage = (page: number) => {
  currentPage.value = page;
};

const visiblePages = computed<(number | string)[]>(() => {
  const total = totalPages.value;
  const current = currentPage.value;
  const pages: (number | string)[] = [];

  if (total <= 7) {
    for (let i = 1; i <= total; i++) pages.push(i);
    return pages;
  }

  pages.push(1);

  if (current <= 4) {
    for (let i = 2; i <= 5; i++) pages.push(i);
    pages.push("...");
  } else if (current >= total - 3) {
    pages.push("...");
    for (let i = total - 4; i <= total - 1; i++) pages.push(i);
  } else {
    pages.push("...");
    pages.push(current - 1);
    pages.push(current);
    pages.push(current + 1);
    pages.push("...");
  }

  pages.push(total);
  return pages;
});

const formatPrice = (p: string) => {
  const n = Number(p);
  if (!isFinite(n)) return p;
  if (n >= 1)
    return n.toLocaleString(undefined, {
      minimumFractionDigits: 2,
      maximumFractionDigits: 2,
    });
  return n.toLocaleString(undefined, {
    minimumFractionDigits: 6,
    maximumFractionDigits: 8,
  });
};
</script>

<style scoped>
.crypto-table {
  background: linear-gradient(180deg, #fffaf0 0%, #fff1d6 100%);
  color: #1a1a1a;
  margin: 20px;
  border-radius: 12px;
  padding: 10px;
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
}

.table-head {
  font-weight: 700;
  border-bottom: 2px solid rgba(156, 117, 48, 0.25);
  padding-bottom: 8px;
  margin-bottom: 6px;
}

.table-row {
  border-bottom: 1px solid rgba(156, 117, 48, 0.08);
  padding: 12px 0;
}

ion-col {
  text-align: center;
}

.rank-col {
  font-weight: 700;
  color: #7a4b00;
}

.name-col .symbol {
  font-weight: 700;
  font-size: 1.05rem;
}

.name-col .fullname {
  font-size: 0.9rem;
  color: #5a5a5a;
}

.price-col .amount {
  font-weight: 700;
  color: #006d77;
}

.controls {
  display: flex;
  justify-content: center;
  margin-top: 12px;
}

.top-row {
  display: flex;
  gap: 12px;
  align-items: center;
  width: 100%;
  max-width: 920px;
  margin: 0 auto;
  padding: 0 12px;
}

.search-input {
  border-radius: 12px;
  --padding-start: 12px;
  width: 70%;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.05);
}

.pagination-controls {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 8px;
  margin: 18px 12px;
  flex-wrap: wrap;
}

ion-button[disabled] {
  opacity: 0.6;
}

.no-results {
  text-align: center;
  padding: 16px;
  color: #666;
}
</style>
