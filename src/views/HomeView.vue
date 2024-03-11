<script setup lang="ts">
import { onMounted, ref } from 'vue'

import InputText from 'primevue/inputtext'
import Button from 'primevue/button'
import MenuComponent from '@/components/MenuComponent.vue'
import { supabase } from '@/config/supabaseClient'
import { useToast } from 'primevue/usetoast';

interface Orders {
  id: number
  value: string
  createdAt: Date
}

interface Setting {
  key: string
  value: string
  createdAt: Date
}

const num = ref()
const orders = ref<Orders[]>([])
const toast = useToast();

const onEnterInput = () => {
  if (num.value.length == 0) return
  post_data(num.value)
  num.value = ''
}

const get_data = async () => {
  let { data, error } = await supabase.from('tb_orders').select('*')
  if (error) {
    toast.add({severity: 'error', summary: 'Error', life: 5000, detail: "error can't get data"})
    return
  }
  orders.value = data as Orders[]
}

const check_limit = async () => {
  let { data, error } = await supabase.from('tb_settings').select('*').eq('key', 'LIMIT').limit(1)
  if (error) {
    toast.add({severity: 'error', summary: 'Error', life: 5000, detail: "error can't get limit"})
    return
  }

  const result = data as Setting[]
  if (result.length == 1) {
    if (orders.value.length < Number.parseInt(result[0].value)) return true
  }
  toast.add({severity: 'warn', summary: 'Warn', life: 5000, detail: `LIMIT ${orders.value.length}/${result[0].value}`})
  return false
}

const post_data = async (data: string) => {
  if (await check_limit()) {
    const { error } = await supabase.from('tb_orders').insert({ value: data })
    if (error) {
      toast.add({severity: 'error', summary: 'Error', life: 5000, detail: "error can't insert data"})
      return
    }
  }
}

const check_update = async () => {
  supabase
    .channel('all-channel')
    .on('postgres_changes', { event: '*', schema: 'public', table: 'tb_orders' }, () => {
      get_data()
    })
    .subscribe()
}

onMounted(() => {
  get_data()
  check_update()
})
</script>

<template>
  <main>
    <MenuComponent :orders="orders" />
    <div class="flex justify-center items-center flex-col h-screen">
      <h1 class="mb-20 bg-green px-20 py-5 text-white rounded-full">{{ orders.length }}</h1>
      <Badge value="2"></Badge>
      <div class="flex gap-2">
        <InputText
          type="text"
          size="large"
          placeholder="Your Number"
          @keyup.enter="onEnterInput"
          v-model="num"
        />
        <Button label="Enter" size="large" @click="onEnterInput" />
      </div>
    </div>
  </main>
</template>
