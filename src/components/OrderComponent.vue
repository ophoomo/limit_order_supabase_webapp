<script setup lang="ts">
import Dialog from 'primevue/dialog'
import Button from 'primevue/button'
import { supabase } from '@/config/supabaseClient'
import DataTable from 'primevue/datatable'
import Column from 'primevue/column'
import dayjs from 'dayjs'
import 'dayjs/locale/th'
import buddhistEra from 'dayjs/plugin/buddhistEra'
import { useToast } from 'primevue/usetoast'
import { useConfirm } from 'primevue/useconfirm'
dayjs.extend(buddhistEra)
dayjs.locale('th')

const props = defineProps(['visible', 'orders'])
const emit = defineEmits(['update-visible'])
const toast = useToast()
const confirm = useConfirm()

const del_data_by_id = async (id: string) => {
  const { error } = await supabase.from('tb_orders').delete().eq('id', id)
  if (error) {
    toast.add({
      severity: 'error',
      summary: 'Error',
      detail: `error can't delete id ${id} data`,
      life: 5000
    })
    return
  }
  // emit('update-visible', false)
}

const confirm_del = async (id: string) => {
  confirm.require({
    message: 'Are you sure you want to delete data?',
    header: 'Confirmation',
    icon: 'pi pi-exclamation-triangle',
    rejectClass: 'p-button-secondary p-button-outlined',
    rejectLabel: 'Cancel',
    acceptLabel: 'Confirm',
    accept: async () => {
      await del_data_by_id(id)
    }
  })
}
</script>

<template>
  <Dialog
    :closable="false"
    v-model:visible="props.visible as boolean"
    modal
    header="Order"
    :style="{ width: '80%' }"
  >
    <DataTable :value="props.orders">
      <Column sortable field="id" header="ID"></Column>
      <Column sortable field="value" header="Value"></Column>
      <Column sortable field="created_at" header="created">
        <template #body="{ data }">
          {{ dayjs(data.created_at).format('DD MMM BB HH:mm:ss') }}
        </template>
      </Column>
      <Column>
        <template #body="{ data }">
          <Button icon="pi pi-trash" severity="danger" @click="confirm_del(data.id)" />
        </template>
      </Column>
    </DataTable>
    <div class="flex justify-end gap-2 mt-10">
      <Button
        type="button"
        label="Cancel"
        severity="secondary"
        @click="emit('update-visible', false)"
      ></Button>
    </div>
  </Dialog>
</template>
