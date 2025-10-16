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
          <h2 class="text-sm leading-none font-medium">Pick a Date</h2>
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
      <template v-else-if="step === 3">
         <div
            class="space-y-4">
            <h2 class="text-sm leading-none font-medium">Pick a Time</h2>
          <ul class="space-y-1">
            <li v-for="slot in times"
              @click="getTime(slot)">
              <span 
                v-if="slot.available" 
                class="block w-full bg-green-500 text-white rounded-md px-2 py-1 text-center cursor-pointer">
                  {{ slot.time }}
              </span>
            </li>
          </ul>
        </div>
      </template>
      <template v-else>
        <div class="space-y-4">
          <p>
            <span class="text-sm leading-none font-medium">submitted Values:</span> {{ values.fullName }} {{ formatDate(values.date) }} {{ values.time }}
          </p>

          <Button 
            @click="checkCalendar"
            class="duration-300 ease-in-out cursor-pointer bg-green-500 hover:bg-green-500/80">
            check Calendar
          </Button>
        </div>
      </template>
    </div>

    <pre>{{ values }}</pre>
    <pre>{{ times }}</pre>
  </div>
</template>

<script setup>
  import { getLocalTimeZone, today } from "@internationalized/date"
  import { ref } from "vue"
  import { Calendar } from "@/components/ui/calendar"

  const dateValue = ref(today(getLocalTimeZone()))

  const step = ref(1)

  const firstName = ref('')
  const lastName = ref('')

  const times = ref([
    {time: '09:00 - 10:00', available: true},
    {time: '11:00 - 12:00', available: true},
    {time: '14:00 - 15:00', available: true}
  ])

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

  function getTime(slot) {
    values.time = slot.time
    slot.available = false
    step.value = 4
  }

  function checkCalendar() {
    step.value = 2
  }
</script>

<style scoped>

</style>