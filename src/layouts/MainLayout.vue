<script setup lang="ts">
import FullScreenLoadingSpinner from "@/components/FullScreenLoadingSpinner.vue";
import Sidebar from "@/layouts/SidebarLayout.vue";
import Header from "@/components/HeaderBar.vue";
import { onMounted, onUnmounted, ref } from "vue";
import { useAuthStore } from "@/stores/auth";
import { useCustomerStore } from "@/stores/customer";
import { useRouter } from "vue-router";
const customerStore = useCustomerStore();
const authStore = useAuthStore();
const router = useRouter();
const tokenValidationInterval = 30 * 1000; // 30s
let interval: number;
let initialized = ref(false);

async function checkIfTokenIsValid() {
  const token = authStore.token;
  await authStore.isTokenValid(token).catch(async () => {
    initialized.value = false;
    await authStore.logout();
    await wait(100);
    router.push("/auth/login");
  });
}

function sleep(ms: number): Promise<void> {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function wait(ms: number) {
  await sleep(ms); // Espera 2 segundos
}

onMounted(async () => {
  await checkIfTokenIsValid();
  await customerStore.initialize();
  initialized.value = true;

  interval = setInterval(async () => {
    await checkIfTokenIsValid();
  }, tokenValidationInterval);
});

onUnmounted(() => {
  clearInterval(interval);
});
</script>
<template>
  <FullScreenLoadingSpinner v-if="!initialized" />
  <main v-else class="flex flex-col h-screen overflow-hidden">
    <header>
      <Header />
    </header>
    <div class="flex h-full p-6 overflow-hidden w-full sm:max-w-4xl mx-auto">
      <div class="flex w-full rounded shadow border-2 border-gray-300 overflow-hidden">
        <div>
          <Sidebar />
        </div>
        <div class="container p-0 h-full">
          <router-view />
        </div>
      </div>
    </div>
  </main>
</template>
