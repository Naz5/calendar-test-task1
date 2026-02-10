<template>
  <div v-if="show" class="event-modal-backdrop" @click.self="close">
    <div class="event-modal">
      <h3>{{ isNew ? 'Add Event' : 'Edit Event' }}</h3>
      <div class="form-group">
        <label for="event-title">Title</label>
        <input id="event-title" v-model="title" maxlength="30" placeholder="Event Title" />
        <small>{{ title.length }} / 30</small>
      </div>
      <div class="form-group">
        <label for="event-color">Color</label>
        <input id="event-color" type="color" v-model="color" />
      </div>
      <div class="modal-actions">
        <button v-if="!isNew" @click="remove" class="btn-delete">Delete</button>
        <button @click="save">Save</button>
        <button @click="close">Cancel</button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  show: Boolean,
  isNew: Boolean,
  eventData: Object
})

const emit = defineEmits(['close', 'save', 'delete'])

const title = ref('')
const color = ref('#3788d8')

watch(
  () => props.show,
  (newVal) => {
    if (newVal) {
      title.value = props.isNew ? '' : props.eventData.title
      color.value = props.isNew ? '#3788d8' : props.eventData.backgroundColor || '#3788d8'
    }
  }
)

const close = () => {
  emit('close')
}

const save = () => {
  if (!title.value) {
    alert('Title is required.')
    return
  }
  emit('save', {
    title: title.value,
    color: color.value
  })
}

const remove = () => {
  emit('delete')
}
</script>

<style scoped>
.event-modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 100;
}

.event-modal {
  background: white;
  padding: 20px;
  border-radius: 8px;
  width: 90%;
  max-width: 400px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.event-modal h3 {
  margin-top: 0;
  margin-bottom: 20px;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
  font-size: 14px;
}

.form-group input[type="text"] {
  width: calc(100% - 20px);
  padding: 8px 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 16px;
}

.form-group small {
  display: block;
  text-align: right;
  font-size: 12px;
  color: #666;
}

.form-group input[type="color"] {
  width: 100%;
  height: 40px;
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 0;
  cursor: pointer;
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  margin-top: 20px;
}

.modal-actions button {
  display: flex;
  align-self: center;
  height: 30px;
  padding: 5px;
}

.modal-actions button.btn-delete {
  background-color: #f44336;
  color: white;
  margin-right: auto;
}
</style>