<template>
  <div class="container mx-auto px-4 py-12">
    <div class="max-w-3xl mx-auto">
      <template v-if="step === 1">
        <form 
          @submit.prevent="getFirstName"
          class="space-y-4">
          <Label 
            for="firstName">
            First Name
          </Label>
          <Input 
            type="text" 
            v-model="firstName" />
          <Button 
            type="submit"
            class="duration-300 ease-in-out cursor-pointer bg-green-500 hover:bg-green-500/80">
            next
          </Button>
        </form>
      </template>
      <template v-else-if="step === 2">
         <form 
          @submit.prevent="getLastName"
          class="space-y-4">
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
      <template v-else>
        <span>submitted Name: {{ values.firstName }} {{ values.lastName }}</span>
      </template>
    </div>
    <pre>{{ values }}</pre>
  </div>
</template>

<script setup>
  const step = ref(1)

  const firstName = ref('')
  const lastName = ref('')

  const values = reactive({})

  function getFirstName() {
    if(firstName.value) {
      values.firstName = firstName.value
      firstName.value = ''
      step.value = 2
    }
  }

  function getLastName() {
    if(lastName.value) {
      values.lastName = lastName.value
      lastName.value = ''
      step.value = 3
    }
  }

</script>

<style scoped>

</style>