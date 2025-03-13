<template>
  <q-page padding>
    <div class="row">
      <div class="col col-12 col-md-8">
        <q-card flat bordered class="q-mb-md">
          <q-card-section>
            <div class="text-h5">System</div>
            <div class="text-subtitle1 q-pt-md">CPU</div>
            <div class="row">
              <div
                v-for="cpu in sortedCPU"
                :key="cpu.node"
                class="column flex-center flex-wrap q-ma-md"
              >
                <span class="text-overline">CPU {{ cpu.node }}</span>
                <q-circular-progress
                  show-value
                  :value="cpu.value"
                  size="50px"
                  color="accent"
                  track-color="grey-3"
                />
              </div>
            </div>
            <div class="text-subtitle1 q-pt-md">Memory</div>
            <q-linear-progress
              size="25px"
              :value="
                parseInt(store.data.metrics.system_memory_used_bytes, 10) /
                parseInt(store.data.metrics.system_memory_total_bytes, 10)
              "
              color="accent"
            >
              <div class="absolute-full flex flex-center">
                <q-badge
                  color="white"
                  text-color="accent"
                  :label="prettyBytes(parseInt(store.data.metrics.system_memory_used_bytes, 10))"
                />
              </div>
              <div class="absolute-full flex justify-end">
                <q-badge
                  color="white"
                  text-color="accent"
                  :label="prettyBytes(parseInt(store.data.metrics.system_memory_total_bytes, 10))"
                />
              </div>
            </q-linear-progress>

            <div class="text-subtitle1 q-pt-md">Disk</div>

            <q-linear-progress
              size="25px"
              :value="
                parseInt(store.data.metrics.system_disk_used_bytes, 10) /
                parseInt(store.data.metrics.system_disk_total_bytes, 10)
              "
              color="accent"
            >
              <div class="absolute-full flex flex-center">
                <q-badge
                  color="white"
                  text-color="accent"
                  :label="prettyBytes(parseInt(store.data.metrics.system_disk_used_bytes, 10))"
                />
              </div>
              <div class="absolute-full flex justify-end">
                <q-badge
                  color="white"
                  text-color="accent"
                  :label="prettyBytes(parseInt(store.data.metrics.system_disk_total_bytes, 10))"
                />
              </div>
            </q-linear-progress>
            <div class="text-subtitle1 q-pt-md">System Network</div>
            <div>
              Received:
              {{ prettyBytes(parseInt(store.data.metrics.system_network_received_bytes, 10)) }}
              Sent:
              {{ prettyBytes(parseInt(store.data.metrics.system_network_sent_bytes, 10)) }}
            </div>
          </q-card-section>
        </q-card>
      </div>
      <q-card flat bordered class="col-12 col-md-3 offset-md-1 q-mb-md">
        <q-card-section>
          <div class="text-h5">Typesense</div>

          <div class="text-subtitle1 q-pt-md">Node</div>

          <div>Protocol: {{ store.loginData?.node.protocol }}</div>
          <div>Host: {{ store.loginData?.node.host }}</div>
          <div>Port: {{ store.loginData?.node.port }}</div>
          <div>Version: {{ store.data.debug.version }}</div>
          <div v-if="Object.hasOwnProperty.call(store.data.debug, 'state')">
            Role:
            {{ store.data.debug.state === 1 ? 'Leader' : 'Follower' }}
          </div>

          <div class="text-subtitle1 q-pt-md">Memory</div>
          <div
            v-for="metric in Object.keys(store.data.metrics).filter((m) => m.includes('typesense'))"
            :key="metric"
          >
            {{ metric.split('_')[2] }} :
            {{
              metric.includes('bytes')
                ? prettyBytes(parseInt(store.data.metrics[metric], 10))
                : store.data.metrics[metric]
            }}
          </div>
          <div class="text-subtitle1 q-pt-md">Stats</div>
          <div v-if="!store.data.features.stats">Stats are not enabled on this node.</div>
          <div v-for="(content, label) in store.data.stats" :key="label">
            <div v-if="isObject(content)">
              {{ label }}
              <div v-for="(value, entry) in content" :key="entry">{{ entry }} : {{ value }}</div>
            </div>
          </div>
        </q-card-section>
      </q-card>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { useNodeStore } from 'src/stores/node';
import { computed, onBeforeUnmount, onMounted, ref } from 'vue';
import prettyBytes from 'pretty-bytes';

const store = useNodeStore();

const refreshInterval = ref<number | undefined>(undefined);

function isObject(obj: unknown) {
  return typeof obj === 'object';
}

onMounted(() => {
  refreshInterval.value = window.setInterval(() => {
    store.refreshServerStatus();
  }, 2000);
});

onBeforeUnmount(() => {
  window.clearInterval(refreshInterval.value);
});

const sortedCPU = computed(() => {
  return Object.entries(store.data.metrics)
    .filter(([key]) => key.includes('cpu'))
    .map(([key, value]) => {
      let node = 0;
      const keyData = key.split('_');
      if (keyData.length > 1 && keyData[1]) {
        node = parseInt(keyData[1].replace('cpu', '')) || 0;
      }
      return {
        node,
        value: parseFloat(value as string),
      };
    })
    .sort((a, b) => a.node - b.node);
});
</script>
