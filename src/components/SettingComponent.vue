<script setup lang="ts">
import Dialog from 'primevue/dialog'
import InputNumber from 'primevue/inputnumber'
import Button from 'primevue/button'
import { onMounted, ref } from 'vue'
import { supabase } from '@/config/supabaseClient'
import { useConfirm } from 'primevue/useconfirm'
import { useToast } from 'primevue/usetoast'

const props = defineProps(['visible'])
const emit = defineEmits(['update-visible'])
const toast = useToast()
const confirm = useConfirm()

const limit = ref(0)

const get_data = async () => {
  const { data, error } = await supabase.from('tb_settings').select()
  if (error) {
    alert('error upsert')
    toast.add({ severity: 'error', summary: 'Error', life: 5000, detail: `error can't get data` })
    return
  }
  data.map((item) => {
    if (item.key == 'LIMIT') limit.value = item.value
  })
}

const onSaveSetting = async () => {
  const { error } = await supabase
    .from('tb_settings')
    .update({ value: limit.value })
    .eq('key', 'LIMIT')
    .select()
  if (error) {
    alert('error upsert')
    toast.add({
      severity: 'error',
      summary: 'Error',
      life: 5000,
      detail: `error can't delete save setting`
    })
    return
  }
  toast.add({ severity: 'success', summary: 'Success', life: 5000, detail: `save setting success` })
  emit('update-visible', false)
}

const clear_data = async () => {
  const { error } = await supabase.from('tb_orders').delete().neq('id', 0)
  if (error) {
    toast.add({ severity: 'error', summary: 'Error', life: 5000, detail: `error can't clear data` })
    return
  }
  toast.add({ severity: 'success', summary: 'Success', life: 5000, detail: `clear data success` })
  emit('update-visible', false)
}

const confirm_clear = () => {
  confirm.require({
    message: 'Are you sure you want to clear data all?',
    header: 'Confirmation',
    icon: 'pi pi-exclamation-triangle',
    rejectClass: 'p-button-secondary p-button-outlined',
    rejectLabel: 'Cancel',
    acceptLabel: 'Confirm',
    accept: () => {
      clear_data()
    }
  })
}

onMounted(() => {
  get_data()
})
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
      <Button
        type="button"
        @click="confirm_clear"
        severity="danger"
        outlined
        label="Clear Order"
        class="w-full"
      />
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
