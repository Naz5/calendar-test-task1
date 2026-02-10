<template>
  <div class="calendar-app">
    <div class="my-custom-toolbar">
      <div class="toolbar-left">
        <h3 class="toolbar-label">Calendar View</h3>
        <div class="btn-group">
        <button @click="api().today()">Today</button>
        <button @click="api().prev()">Back</button>
        <button @click="api().next()">Next</button>
        </div>
      </div>

      <div class="toolbar-center">
        <h2>{{ currentTitle }}</h2>
      </div>

      <div class="toolbar-right">
        <button
          :class="{ active: currentView === 'dayGridMonth' }"
          @click="changeView('dayGridMonth')"
        >
          Month
        </button>
        <button
          :class="{ active: currentView === 'timeGridWeek' }"
          @click="changeView('timeGridWeek')"
        >
          Week
        </button>
        <button
          :class="{ active: currentView === 'timeGridDay' }"
          @click="changeView('timeGridDay')"
        >
          Day
        </button>
        <button
          :class="{ active: currentView === 'listMonth' }"
          @click="changeView('listMonth')"
        >
          Agenda
        </button>
      </div>
    </div>

    <FullCalendar ref="fullCalendar" :options="calendarOptions" />

    <EventModal
      :show="eventModal.show"
      :is-new="eventModal.isNew"
      :event-data="eventModal.eventData"
      @close="closeModal"
      @save="saveEvent"
      @delete="deleteEvent"
    />
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import FullCalendar from '@fullcalendar/vue3'
import dayGridPlugin from '@fullcalendar/daygrid'
import timeGridPlugin from '@fullcalendar/timegrid'
import listPlugin from '@fullcalendar/list'
import interactionPlugin from '@fullcalendar/interaction'
import EventModal from './components/EventModal.vue'

const fullCalendar = ref(null)
const currentTitle = ref('')
const currentView = ref('dayGridMonth')

const calendarEvents = ref([
  {
    id: '1',
    title: 'Team Meeting',
    start: new Date().toISOString().split('T')[0] + 'T10:00:00',
    end: new Date().toISOString().split('T')[0] + 'T12:00:00',
    backgroundColor: '#2196F3',
    borderColor: '#2196F3'
  },
  {
    id: '2',
    title: 'Lunch',
    start: new Date().toISOString().split('T')[0] + 'T13:00:00',
    backgroundColor: '#4CAF50',
    borderColor: '#4CAF50'
  },
  {
    id: '3',
    title: 'Project Deadline',
    start: new Date(new Date().setDate(new Date().getDate() + 1)).toISOString().split('T')[0],
    allDay: true,
    backgroundColor: '#f44336',
    borderColor: '#f44336'
  }
])

const eventModal = ref({
  show: false,
  isNew: true,
  eventData: null
})

const changeView = (viewName) => {
  currentView.value = viewName
  api().changeView(viewName)
}

const api = () => fullCalendar.value?.getApi()

const closeModal = () => {
  eventModal.value.show = false
}

const saveEvent = (payload) => {
  if (eventModal.value.isNew) {
    // Add new event
    calendarEvents.value.push({
      id: String(Date.now()),
      title: payload.title,
      start: eventModal.value.eventData.dateStr,
      allDay: eventModal.value.eventData.allDay,
      backgroundColor: payload.color,
      borderColor: payload.color
    })
  } else {
    // Edit existing event
    const eventId = eventModal.value.eventData.id
    const eventIndex = calendarEvents.value.findIndex((e) => e.id === eventId)
    if (eventIndex !== -1) {
      const existingEvent = calendarEvents.value[eventIndex]
      calendarEvents.value[eventIndex] = {
        ...existingEvent,
        title: payload.title,
        backgroundColor: payload.color,
        borderColor: payload.color
      }
    }
  }
  closeModal()
}

const deleteEvent = () => {
  if (confirm('Are you sure you want to delete this event?')) {
    const eventId = eventModal.value.eventData.id
    calendarEvents.value = calendarEvents.value.filter((e) => e.id !== eventId)
    closeModal()
  }
}

const handleDateClick = (arg) => {
  eventModal.value = {
    show: true,
    isNew: true,
    eventData: arg
  }
}

const handleEventClick = (arg) => {
  eventModal.value = {
    show: true,
    isNew: false,
    eventData: arg.event
  }
}

const handleEventDropOrResize = (arg) => {
  const eventId = arg.event.id
  const eventIndex = calendarEvents.value.findIndex((e) => e.id === eventId)
  if (eventIndex !== -1) {
    const existingEvent = calendarEvents.value[eventIndex]
    calendarEvents.value[eventIndex] = {
      ...existingEvent,
      start: arg.event.start,
      end: arg.event.end,
      allDay: arg.event.allDay
    }
  }
}

const calendarOptions = reactive({
  plugins: [dayGridPlugin, timeGridPlugin, listPlugin, interactionPlugin],
  headerToolbar: false,
  initialView: 'dayGridMonth',
  datesSet: (info) => {
    currentTitle.value = info.view.title
  },
  events: calendarEvents,
  editable: true,
  selectable: true,
  navLinks: true,
  dateClick: handleDateClick,
  eventClick: handleEventClick,
  eventDrop: handleEventDropOrResize,
  eventResize: handleEventDropOrResize
})
</script>

<style scoped>
.calendar-app {
  font-family: var(--font-main), serif;
  font-size: 18px;
  letter-spacing: 0;
  padding: 20px;
}

.my-custom-toolbar {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  margin-bottom: 20px;
}

.my-custom-toolbar button {
  font-size: 13px;
  height: 32px;
  border-radius: 0;
  padding: 8px 16px;
  border: 1px solid var(--border-color);
  font-family: var(--font-main), serif;
  transition: all 0.2s;
}

.my-custom-toolbar button:first-child {
  margin-left: 0;
  border-top-left-radius: 4px;
  border-bottom-left-radius: 4px;
}

.my-custom-toolbar button:last-child {
  border-top-right-radius: 4px;
  border-bottom-right-radius: 4px;
}

.my-custom-toolbar button.active {
  color: var(--primary-blue);
  z-index: 1;
}

.my-custom-toolbar button:hover {
  z-index: 2;
  border-color: var(--primary-blue);
  color: var(--primary-blue);
}

.toolbar-label {
  font-size: 18px;
  margin-bottom: 12px;
}

.btn-group button {
  padding: 6px 12px;
  border: 1px solid #ddd;
  background: #fff;
  cursor: pointer;
}



.toolbar-center {
  margin-bottom: 0;
  h2 {
    color: #4D4F5C;
    font-size: 18px;
  }
}

.toolbar-center h2 {
    color: #4D4F5C;
}

.toolbar-right {
  align-self: center;
  margin-top: 5px;
}

:deep(.fc-theme-standard .fc-col-header-cell) {
  background-color: var(--bg-light);
  padding: 12px 0;
  border: 0;
}

:deep(.fc-col-header-cell-cushion) {
  color: #A3A6B4;
  font-size: 11px;
  font-weight: 700;
  text-transform: uppercase;
  text-decoration: none !important;
  letter-spacing: 0;
  border: 0px;
}

:deep(.fc-day-today) {
  background-color: var(--bg-light) !important;
}

:deep(.fc .fc-daygrid-day.fc-day-today) {
  background-color: var(--bg-light)  !important;
}
</style>
