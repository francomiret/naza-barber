<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import AppointmentForm from './components/AppointmentForm.vue';
import AppointmentList from './components/AppointmentList.vue';

interface Appointment {
  id: number;
  customerName: string;
  date: string;
  time: string;
  services: string[];
}

const appointments = ref<Appointment[]>([]);
const isCreateDialogOpen = ref(false);

onMounted(() => {
  try {
    const storedAppointments = localStorage.getItem('appointments');
    if (storedAppointments) {
      appointments.value = JSON.parse(storedAppointments);
    }
  } catch (error) {
    console.error('Error loading appointments:', error);
  }
});

const addAppointment = (appointment: Omit<Appointment, 'id'>) => {
  const newAppointment = {
    ...appointment,
    id: Date.now(),
  };
  appointments.value.push(newAppointment);
  saveAppointments();
  isCreateDialogOpen.value = false;
};

const updateAppointment = (updatedAppointment: Appointment) => {
  const index = appointments.value.findIndex(
    (a) => a.id === updatedAppointment.id
  );
  if (index !== -1) {
    appointments.value[index] = updatedAppointment;
    saveAppointments();
  }
};

const deleteAppointment = (id: number) => {
  appointments.value = appointments.value.filter((a) => a.id !== id);
  saveAppointments();
};

const saveAppointments = () => {
  try {
    localStorage.setItem('appointments', JSON.stringify(appointments.value));
  } catch (error) {
    console.error('Error saving appointments:', error);
  }
};

const sortedAppointments = computed(() => {
  return appointments.value.sort((a, b) => {
    const dateA = new Date(`${a.date}T${a.time}`);
    const dateB = new Date(`${b.date}T${b.time}`);
    return dateA.getTime() - dateB.getTime();
  });
});

const groupedAppointments = computed(() => {
  const groups: { [key: string]: Appointment[] } = {};
  sortedAppointments.value.forEach((appointment) => {
    if (!groups[appointment.date]) {
      groups[appointment.date] = [];
    }
    groups[appointment.date].push(appointment);
  });
  return groups;
});
</script>

<template>
  <v-app>
    <v-app-bar color="primary" dark app>
      <v-app-bar-title>Naza Barber</v-app-bar-title>
      <v-spacer></v-spacer>
      <v-btn @click="isCreateDialogOpen = true" variant="outlined">
        Crear Turno
      </v-btn>
    </v-app-bar>

    <v-main>
      <v-container>
        <v-row>
          <v-col cols="12">
            <AppointmentList
              :grouped-appointments="groupedAppointments"
              @update-appointment="updateAppointment"
              @delete-appointment="deleteAppointment"
            />
          </v-col>
        </v-row>
      </v-container>
    </v-main>

    <v-dialog v-model="isCreateDialogOpen" max-width="500px">
      <v-card>
        <v-card-title>Crear Nuevo Turno</v-card-title>
        <v-card-text>
          <AppointmentForm @add-appointment="addAppointment" />
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-app>
</template>
