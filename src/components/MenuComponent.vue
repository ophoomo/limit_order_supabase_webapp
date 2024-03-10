<script setup lang="ts">
import Menubar from 'primevue/menubar'
import { ref } from 'vue';
import SettingComponent from './SettingComponent.vue';
import OrderComponent from './OrderComponent.vue';

const props = defineProps(['orders'])

const visible_setting = ref(false);
const visible_order = ref(false);

const items = ref([
  {
    label: 'Order',
    icon: 'pi pi-list',
    command: () => visible_order.value = true,
  },
  {
    label: 'Setting',
    icon: 'pi pi-cog',
    command: () => visible_setting.value = true,
  }
])
</script>

<template>
  <div class="card">
    <Menubar :model="items">
      <template #item="{ item, props, hasSubmenu, root }">
        <a v-ripple class="flex align-items-center" v-bind="props.action">
          <span :class="item.icon" />
          <span class="ml-2">{{ item.label }}</span>
          <Badge
            v-if="item.badge"
            :class="{ 'ml-auto': !root, 'ml-2': root }"
            :value="item.badge"
          />
          <span
            v-if="item.shortcut"
            class="ml-auto border-1 surface-border border-round surface-100 text-xs p-1"
            >{{ item.shortcut }}</span
          >
          <i
            v-if="hasSubmenu"
            :class="[
              'pi pi-angle-down',
              { 'pi-angle-down ml-2': root, 'pi-angle-right ml-auto': !root }
            ]"
          ></i>
        </a>
      </template>
    </Menubar>
  </div>
  <SettingComponent @update-visible="(i) => visible_setting = i" :visible="visible_setting" />
  <OrderComponent :orders="props.orders" @update-visible="(i) => visible_order = i" :visible="visible_order" />
</template>
