<template>
  <div class="container mx-auto px-4 py-12">
    <div class="max-w-3xl mx-auto">
      <Transition name="fade" mode="out-in">
        <div :key="step">
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
              <p v-if="errorMessage" 
                class="text-red-500">
                {{ errorMessage }}
              </p>
            </form>
          </template>

          <template v-else-if="step === 2">
            <div
              class="space-y-4 w-fit">
              <h2 class="text-sm leading-none font-medium">Pick a Date</h2>
              <form @submit.prevent="getDate">
                <Calendar 
                  v-model="dateValue" 
                  :weekday-format="'short'" 
                  class="rounded-md border" />
              </form>
            </div>
          </template>

          <template v-else-if="step === 3">
            <div class="space-y-4">
              <h2 class="text-sm leading-none font-medium">Pick a Time</h2>
              <ul class="space-y-1">
                <li v-for="slot in times" :key="slot.times_id.id" @click="getTime(slot)">
                  <span v-if="slot.time_available" 
                        class="block w-full bg-green-500 text-white rounded-md px-2 py-1 text-center cursor-pointer">
                    {{ slot.times_id.time }}
                  </span>
                </li>
              </ul>
            </div>
          </template>

          <template v-else-if="step === 4">
            <div class="space-y-4">
              <div>
                <h2 class="text-sm leading-none font-medium mb-2">Your appointment:</h2>
                <span class="block">Name: {{ values.name }}</span>
                <span class="block">Date: {{ formatDate(values.date) }}</span>
                <span class="block">Time: {{ values.time }}</span>
              </div>

              <Button 
                @click="submit"
                class="duration-300 ease-in-out cursor-pointer bg-green-500 hover:bg-green-500/80">
                confirm
              </Button>
            </div>
          </template>

          <template v-else>
            <div>
              <p class="bg-green-400 text-white w-fit mx-auto px-3 py-2 text-xl">Your appointment has been successfully booked.</p>
            </div>
          </template>
        </div>
      </Transition>

      <Button
        @click="goBack()"
        class="duration-300 ease-in-out cursor-pointer bg-red-500 hover:bg-red-500/80 mt-2">
        back
      </Button>

      <div class="max-w-3xl mx-auto">
        <ul>
          <li v-for="item in bookings">
            {{ item.name }} {{ item.time_available }}
          </li>
        </ul>
      </div>
    </div>

    <pre>{{times}}</pre>
  </div>
</template>

<script setup>
  import { getLocalTimeZone, today } from "@internationalized/date"
  import { ref, reactive, onMounted } from "vue"
  import { Calendar } from "@/components/ui/calendar"

  const { createItems, getItems, updateItem } = useDirectusItems()

  const step = ref(1)
  const firstName = ref('')
  const lastName = ref('')
  const errorMessage = ref('')

  const dateValue = ref(today(getLocalTimeZone()))
  const selectedDate = ref(null)
  const selectedTime = ref(null)
  const times = ref([])

  const values = reactive({
    name: '',
    date: '',
    time: ''
  })

  const bookings = ref([])
  const dates = ref([])

  // --- Fetch dates and bookings ---
  const fetchDates = async () => {
    const data = await getItems({
      collection: "dates",
      params: {
        fields: [
          '*',
          'times_available.*',
          'times_available.times_id.*'
        ],
      },
    })

    dates.value = data.map(d => ({
      ...d,
      has_available_times: d.times_available.some(t => t.time_available)
    }))
  }

  const fetchBookings = async () => {
    bookings.value = await getItems({
      collection: "bookings",
      params: { fields: ['*'] }
    })
  }

  function goBack() {
    if (step.value > 1) step.value--
  }

  function getName() {
    if (!firstName.value || !lastName.value) {
      errorMessage.value = 'Please enter your full Name.'
      setTimeout(() => (errorMessage.value = ''), 3000)
      return
    }
    values.name = `${firstName.value} ${lastName.value}`
    firstName.value = ''
    lastName.value = ''
    step.value = 2
  }

  function getDate() {
    if (!dateValue.value) return

    const jsDate = new Date(dateValue.value.year, dateValue.value.month - 1, dateValue.value.day)
    const dateString = `${jsDate.getFullYear()}-${String(jsDate.getMonth()+1).padStart(2,'0')}-${String(jsDate.getDate()).padStart(2,'0')}`

    selectedDate.value = dates.value.find(d => d.date === dateString && d.day_available)

    if (!selectedDate.value || !selectedDate.value.has_available_times) {
      errorMessage.value = 'No available times for this date.'
      setTimeout(() => (errorMessage.value = ''), 3000)
      return
    }

    times.value = selectedDate.value.times_available.filter(t => t.time_available)

    values.date = dateString
    step.value = 3
  }

  function getTime(slot) {
    selectedTime.value = slot
    values.time = slot.times_id.time
    step.value = 4
  }

  const submit = async () => {
    try {
      if (!selectedTime.value?.id) throw new Error('No time selected')

      await updateItem({
        collection: 'date_times',
        id: selectedTime.value.id,
        item: { time_available: false }
      })

      await createItems({
        collection: 'bookings',
        items: [{
          name: values.name,
          date: values.date,
          time: values.time
        }]
      })

      await fetchBookings()
      await fetchDates()

      const updatedDate = dates.value.find(d => d.id === selectedTime.value.dates_id)

      const anyAvailable = updatedDate?.times_available.some(t => t.time_available)

      if (!anyAvailable) {
        await updateItem({
          collection: 'dates',
          id: updatedDate.id,
          item: { day_available: false }
        })

        updatedDate.day_available = false
      }

      selectedDate.value = updatedDate
      times.value = selectedDate.value?.times_available.filter(t => t.time_available) || []

      step.value = 5
      setTimeout(() => step.value = 1, 3000)

    } catch (err) {
      console.error(err)
      errorMessage.value = 'Something went wrong. Please try again.'
      step.value = 3
    }
  }

  onMounted(async () => {
    await fetchBookings()
    await fetchDates()
  })
</script>

<style scoped>
  .fade-enter-active,
  .fade-leave-active {
    transition: all 0.3s;
  }
  .fade-enter-from,
  .fade-leave-to {
    opacity: 0;
  }
</style>