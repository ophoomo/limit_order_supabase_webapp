<script setup lang="ts">
import { onMounted, ref } from 'vue'

import InputText from 'primevue/inputtext'
import Button from 'primevue/button'
import MenuComponent from '@/components/MenuComponent.vue'
import { supabase } from '@/config/supabaseClient'

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

const onEnterInput = () => {
  if (num.value.length == 0) return
  post_data(num.value)
  num.value = ''
}

const get_data = async () => {
  let { data, error } = await supabase.from('tb_orders').select('*')
  if (error) {
    alert('error get data')
    return
  }
  orders.value = data as Orders[]
}

const check_limit = async () => {
  let { data, error } = await supabase.from('tb_settings').select('*').eq('key', 'LIMIT').limit(1)
  if (error) {
    alert('error limit')
    return
  }

  const result = data as Setting[]
  if (result.length == 1) {
    if (orders.value.length < Number.parseInt(result[0].value)) return true
  }
  return false
}

const post_data = async (data: string) => {
  if (await check_limit()) {
    const { error } = await supabase.from('tb_orders').insert({ value: data })
    if (error) {
      alert('error insert')
      return
    }
    get_data()
  }
}



onMounted(() => {
  get_data()
  setInterval(() => get_data(), 10000)
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
