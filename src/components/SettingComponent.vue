<script setup lang="ts">
import Dialog from 'primevue/dialog'
import InputNumber from 'primevue/inputnumber'
import Button from 'primevue/button'
import { ref } from 'vue'
import { supabase } from '@/config/supabaseClient';

const props = defineProps(['visible'])
const emit = defineEmits(['update-visible'])

const limit = ref(0)

const onSaveSetting = async () => {
  const { error } = await supabase
  .from('tb_settings')
  .update({ value: limit.value })
  .eq("key", "LIMIT")
  .select()
  if (error) {
    alert("error upsert")
    return
  }
  emit('update-visible', false)
}

const clear_data = async () => {
  const { error } = await supabase.from('tb_orders').delete().neq('id', 0)
  if (error) {
    alert("error");
  }
}
</script>

<template>
  <Dialog
    :closable="false"
    v-model:visible="props.visible as boolean"
    modal
    header="Setting"
    :style="{ width: '25rem' }"
  >
    <div class="flex align-items-center gap-3 mb-3">
      <label for="limit" class="font-semibold w-6rem">Limit</label>
      <InputNumber v-model="limit" :min="0" id="limit" class="flex-auto" autocomplete="off" />
    </div>
    <div class="mt-10 w-full">
      <Button type="button" @click="clear_data" severity="danger" outlined label="Clear Order" class="w-full" />
    </div>
    <div class="flex justify-end gap-2 mt-10">
      <Button
        type="button"
        label="Cancel"
        severity="secondary"
        @click="emit('update-visible', false)"
      ></Button>
      <Button type="button" label="Save" @click="onSaveSetting"></Button>
    </div>
  </Dialog>
</template>
