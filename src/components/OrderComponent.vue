<script setup lang="ts">
import Dialog from 'primevue/dialog'
import Button from 'primevue/button'
import { supabase } from '@/config/supabaseClient'
import DataTable from 'primevue/datatable'
import Column from 'primevue/column'
import dayjs from 'dayjs'
import 'dayjs/locale/th'
import buddhistEra from 'dayjs/plugin/buddhistEra'
dayjs.extend(buddhistEra)
dayjs.locale('th')

const props = defineProps(['visible', 'orders'])
const emit = defineEmits(['update-visible'])

const del_data_by_id = async (id: string) => {
  const { error } = await supabase.from('tb_orders').delete().eq('id', id)
  if (error) {
    alert('error delete')
    return
  }
  emit('update-visible', false)
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
          <Button icon="pi pi-trash" severity="danger" @click="del_data_by_id(data.id)" />
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
