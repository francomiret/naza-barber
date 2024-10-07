<script setup lang="ts">
import { ref, computed } from 'vue';

interface Appointment {
  id: number;
  customerName: string;
  date: string;
  time: string;
  services: string[];
}

const props = defineProps<{
  groupedAppointments: { [key: string]: Appointment[] };
}>();

const emit = defineEmits(['update-appointment', 'delete-appointment']);

const editingAppointment = ref<Appointment | null>(null);
const isDialogOpen = ref(false);

const startEditing = (appointment: Appointment) => {
  editingAppointment.value = { ...appointment };
  isDialogOpen.value = true;
};

const saveEdit = () => {
  if (editingAppointment.value) {
    emit('update-appointment', editingAppointment.value);
    closeDialog();
  }
};

const closeDialog = () => {
  editingAppointment.value = null;
  isDialogOpen.value = false;
};

const deleteAppointment = (id: number) => {
  emit('delete-appointment', id);
};

const formatDate = (dateString: string) => {
  const options: Intl.DateTimeFormatOptions = {
    weekday: 'long',
    year: 'numeric',
    month: 'long',
    day: 'numeric',
  };

  // Asegura que la fecha se maneje como local de Argentina, añadiendo una hora ficticia si es necesario
  const dateParts = dateString.split('-');
  const date = new Date(
    Number(dateParts[0]),
    Number(dateParts[1]) - 1,
    Number(dateParts[2]),
    12
  ); // Set at noon to avoid timezone shifts

  return date.toLocaleDateString('es-AR', options);
};
const today = new Date();
today.setHours(0, 0, 0, 0);

const filteredAppointments = computed(() => {
  const filtered: { [key: string]: Appointment[] } = {};
  Object.entries(props.groupedAppointments).forEach(([date, appointments]) => {
    const appointmentDate = new Date(date);
    if (appointmentDate >= today) {
      filtered[date] = appointments;
    }
  });
  return filtered;
});
</script>

<template>
  <v-card title="Turnos">
    <v-list>
      <template
        v-for="(appointments, date) in filteredAppointments"
        :key="date"
      >
        <v-list-subheader>{{ formatDate(date) }}</v-list-subheader>
        <v-list-item v-for="appointment in appointments" :key="appointment.id">
          <v-list-item-content>
            <v-list-item-title>{{
              appointment.customerName
            }}</v-list-item-title>
            <v-list-item-subtitle>
              <b>
                {{ appointment.time }}
              </b>
              -
              <span v-for="service in appointment.services">
                <v-chip size="x-small" style="margin-right: 5px">
                  {{ service }}
                </v-chip>
              </span>
            </v-list-item-subtitle>
          </v-list-item-content>
          <template v-slot:append>
            <v-btn
              @click="startEditing(appointment)"
              icon="mdi-pencil"
              color="primary"
              density="comfortable"
              size="small"
              style="margin: 5px"
            ></v-btn>
            <v-btn
              @click="deleteAppointment(appointment.id)"
              icon="mdi-delete"
              color="error"
              density="comfortable"
              size="small"
              style="margin: 5px"
            ></v-btn>
          </template>
        </v-list-item>
        <v-divider v-if="appointments.length > 0"></v-divider>
      </template>
    </v-list>

    <v-dialog v-model="isDialogOpen" max-width="500px">
      <v-card v-if="editingAppointment">
        <v-card-title>Editar Turno</v-card-title>
        <v-card-text>
          <v-text-field
            v-model="editingAppointment.customerName"
            label="Nombre del Cliente"
          ></v-text-field>
          <v-text-field
            v-model="editingAppointment.date"
            label="Fecha"
            type="date"
          ></v-text-field>
          <v-text-field
            v-model="editingAppointment.time"
            label="Hora"
            type="time"
          ></v-text-field>
          <v-select
            v-model="editingAppointment.services"
            :items="['Corte', 'Barba', 'Tintura']"
            label="Servicios"
            multiple
            chips
          ></v-select>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" @click="saveEdit">Guardar</v-btn>
          <v-btn color="error" @click="closeDialog">Cancelar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-card>
</template>
