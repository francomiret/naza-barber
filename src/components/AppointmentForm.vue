<script setup lang="ts">
import { ref } from 'vue';

const emit = defineEmits(['add-appointment']);

const customerName = ref('');
const date = ref('');
const time = ref('');
const services = ref<string[]>([]);

const availableServices = ['Corte', 'Barba', 'Tintura'];

const formError = ref('');

const submitForm = () => {
  formError.value = '';
  if (
    customerName.value &&
    date.value &&
    time.value &&
    services.value.length > 0
  ) {
    emit('add-appointment', {
      customerName: customerName.value,
      date: date.value,
      time: time.value,
      services: services.value,
    });
    resetForm();
  } else {
    formError.value = 'Por favor, complete todos los campos requeridos.';
  }
};

const resetForm = () => {
  customerName.value = '';
  date.value = '';
  time.value = '';
  services.value = [];
  formError.value = '';
};
</script>

<template>
  <v-form @submit.prevent="submitForm">
    <v-text-field
      v-model="customerName"
      label="Nombre del Cliente"
      required
    ></v-text-field>
    <v-text-field
      v-model="date"
      label="Fecha"
      type="date"
      required
    ></v-text-field>
    <v-text-field
      v-model="time"
      label="Hora"
      type="time"
      required
    ></v-text-field>
    <v-select
      v-model="services"
      :items="availableServices"
      label="Servicios"
      multiple
      chips
      required
    ></v-select>
    <v-alert v-if="formError" type="error" dense>{{ formError }}</v-alert>
    <v-btn type="submit" color="primary" block>Agregar Turno</v-btn>
  </v-form>
</template>
