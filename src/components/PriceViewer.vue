<script setup lang="ts">
import { ref, onMounted, onUnmounted, computed } from "vue";
import InputMoney from "./InputMoney.vue";

interface PriceData {
  ind: number;
  last: number;
}

const connectionStatus = ref("Disconnected");
const prices = ref<PriceData | null>(null);
const quote = ref(1000);
const fee = ref(0.03);
const receiptedFiat = computed(() => {
  if (!prices.value) return null;
  return (prices.value.ind / quote.value) * (1 - fee.value);
});
const paidFiatFee = computed(() => {
  if (!prices.value) return null;
  return (prices.value.ind / quote.value) * (fee.value);
});

const receiptedSats = computed(() => {
  if (!prices.value) return null;

  return (quote.value / prices.value.ind) * (1 - fee.value);
});
const paidSatsFee = computed(() => {
  if (!prices.value) return null;
  return (quote.value / prices.value.ind) * (fee.value);
});

let ws: WebSocket | null = null;

const connectWebSocket = () => {
  ws = new WebSocket("wss://api.sideswap.io/json-rpc-ws");

  ws.onopen = () => {
    connectionStatus.value = "Connected";
    // Send initial request
    ws?.send(
      JSON.stringify({
        id: 5,
        method: "load_prices",
        params: {
          asset:
            "02f22f8d9c76ab41661a2729e4752e2c5d1a263012141b86ea98af5472df5189",
        },
      })
    );
  };

  ws.onmessage = (event) => {
    const data = JSON.parse(event.data);
    if (data.method === "update_prices") {
      prices.value = {
        ind: data.params.ind,
        last: data.params.last,
      };
    }
  };

  ws.onerror = (error) => {
    connectionStatus.value = "Error: " + error;
  };

  ws.onclose = () => {
    connectionStatus.value = "Disconnected";
  };
};

onMounted(() => {
  connectWebSocket();
});

onUnmounted(() => {
  ws?.close();
});
</script>

<template>
  <div class="price-viewer">
    <h2>Preços p2p</h2>
    <div class="status">Connection Status: {{ connectionStatus }}</div>

    <div v-if="prices" class="prices">
      <div class="price-item">
        <span class="label">Cotação:</span>
        <span class="value">1BTC = R$ {{ prices?.ind?.toFixed(2) }}</span>
      </div>

      <div class="input-section">
        <InputMoney v-model="quote" />
      </div>
      <div class="price-item">
        <span class="label">Recebido com taxa: (3%)</span>
        <div style="display: flex; flex-direction: column; align-items: end">
          <span class="value">R${{ receiptedFiat }}</span>
          <span class="value">{{ receiptedSats }}btc</span>
        </div>
      </div>
      <div class="price-item">
        <span class="label">Taxa paga:</span>
        <div style="display: flex; flex-direction: column; align-items: end">
          <span class="value">R${{ paidFiatFee }}</span>
          <span class="value">{{ paidSatsFee }}btc</span>
        </div>
      </div>
    </div>
    <div v-else class="loading">Waiting for price data...</div>
  </div>
</template>

<style scoped>
.price-viewer {
  padding: 2rem;
  background: #f5f5f5;
  border-radius: 8px;
  max-width: 400px;
  margin: 0 auto;
}

h2 {
  color: #2c3e50;
  margin-bottom: 1rem;
}

.status {
  margin-bottom: 1rem;
  padding: 0.5rem;
  background: #e9ecef;
  border-radius: 4px;
}

.prices {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.price-item {
  display: flex;
  justify-content: space-between;
  padding: 0.5rem;
  background: white;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.label {
  font-weight: bold;
  color: #666;
}

.value {
  color: #2c3e50;
}

.loading {
  text-align: center;
  color: #666;
  padding: 1rem;
}

.input-section {
  margin-top: 1rem;
  padding: 1rem;
  background: white;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
</style>
