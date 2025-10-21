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

      <div class="max-w-3xl mx-auto">
        <ul>
          <li v-for="item in bookings">
            {{ item.name }}
            <ul>
              <li v-for="table in item.table">
                {{ table.test_table_id.relationship }}
              </li>
            </ul>
          </li>
        </ul>
      </div>
    </div>
    {{ values }}
  </div>
</template>

<script setup lang="ts">
  import { getLocalTimeZone, today } from "@internationalized/date"
  import { ref } from "vue"
  import { Calendar } from "@/components/ui/calendar"

  const dateValue = ref(today(getLocalTimeZone()))

  const step = ref<number>(1)

  const firstName = ref<string>('')
  const lastName = ref<string>('')

  const errorMessage = ref<string>('')

  interface TimeSlots {
    time: string,
    available: boolean
  }

  const times = ref<TimeSlots[]>([
    {time: '09:00 - 10:00', available: true},
    {time: '11:00 - 12:00', available: true},
    {time: '14:00 - 15:00', available: true}
  ])

   interface Values {
    name: string;
    date: string;
    time: string;
  }

  const values = reactive<Partial<Values>>({})

  interface Bookings {
    name: string;
    date: string;
    time: string;
    table: any[];
  }

  const bookings = ref<Bookings[]>([])

  const fetchBookings = async () => {
    bookings.value = await getItems<Bookings>({
      collection: "bookings",
      params: {
        fields: ['*', '*.*', 'table.test_table_id.*'],
        // filter: {
        //   name: 'test name'
        // },
      },
    });
  }

  function getName(): void {
    if(!firstName.value || !lastName.value) {
      errorMessage.value = 'Please enter your full Name.'

      setTimeout(() => {
        errorMessage.value =''
      }, 3000)
    }
    else {
      values.name = `${firstName.value} ${lastName.value}`
      firstName.value = ''
      lastName.value = ''
      step.value = 2
    }
  }

  function getDate(): void {
    if (dateValue.value) {
      values.date = dateValue.value.toString()
      step.value = 3
    }
  }

  function getTime(slot: TimeSlots): void {
    values.time = slot.time
    slot.available = false
    step.value = 4
  }

  const { createItems, getItems } = useDirectusItems()

  const submit = async (): Promise<void> => {
    try {
      const items: Values[] = [
        {
          name: values.name,
          date: values.date,
          time: values.time
        },
      ]

      await createItems({
        collection: 'bookings',
        items,
      })

      console.log('Articles created successfully!')
    } catch (error) {
      console.error('Error creating articles:', error)
    }

    step.value = 5

    fetchBookings()

    setTimeout(() => {
      step.value = 1
    }, 3000)
  }

  onMounted(() => {
    fetchBookings()
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