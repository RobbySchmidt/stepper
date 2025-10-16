<template>
  <div class="container mx-auto px-4 py-12">
    <div class="max-w-3xl mx-auto">
      <template v-if="step === 1">
        <form 
          @submit.prevent="getName"
          class="space-y-4">
          <Label 
            for="firstName">
            First Name
          </Label>
          <Input 
            type="text" 
            v-model="firstName" />

          <Label 
            for="lastName">
            Last Name
          </Label>
          <Input 
            type="text" 
            v-model="lastName" />
          <Button 
            type="submit"
            class="duration-300 ease-in-out cursor-pointer bg-green-500 hover:bg-green-500/80">
            next
          </Button>
        </form>
      </template>
      <template v-else-if="step === 2">
         <div
          class="space-y-4 w-fit">
          <Calendar 
            v-model="dateValue" 
            :weekday-format="'short'" 
            class="rounded-md border" />
          <Button 
            @click="getDate"
            class="duration-300 ease-in-out cursor-pointer bg-green-500 hover:bg-green-500/80">
            next
          </Button>
        </div>
      </template>
      <template v-else>
        <span>submitted Values: {{ values.fullName }} {{ formatDate(values.date) }}</span>
      </template>
    </div>
  </div>
</template>

<script setup lang="ts">
  import type { DateValue } from "@internationalized/date"
  import { getLocalTimeZone, today } from "@internationalized/date"
  import { ref } from "vue"
  import { Calendar } from "@/components/ui/calendar"

  const dateValue = ref(today(getLocalTimeZone())) as Ref<DateValue>

  const step = ref(1)

  const firstName = ref('')
  const lastName = ref('')

  const values = reactive({})

  function getName() {
    if(firstName.value && lastName.value) {
      values.fullName = firstName.value + ' ' + lastName.value
      firstName.value = ''
      lastName.value = ''
      step.value = 2
    }
  }

  function getDate() {
    if(dateValue.value) {
      values.date = dateValue.value
      step.value = 3
    }
  }
</script>

<style scoped>

</style>