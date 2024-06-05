<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'
import { Icon } from '@iconify/vue'

const showModal = ref(false)
const newNote = ref('')
const isEditing = ref()
const notes = ref([])
const textAreaBorderColor = ref('#333')
const hasScrolled = ref(false)

onMounted(() => {
  window.addEventListener('scroll', handleScroll);
  
  const storedData = localStorage.getItem('notes')
  if (storedData) {
    notes.value = JSON.parse(storedData)
  }
});

onBeforeUnmount(() => {
  window.removeEventListener('scroll', handleScroll);
});

function handleScroll() {
  hasScrolled.value = window.scrollY > 50
}

function checkTextArea() {
  if (newNote.value === '') {
    textAreaBorderColor.value = '#E53935'
    return false
  } else {
    textAreaBorderColor.value = '#333'
    return true
  }
}

function openModal() {
  showModal.value = true

  document.body.style.overflow = 'hidden'
}

function closeModal() {
  showModal.value = false
  newNote.value = ""
  textAreaBorderColor.value = '#333'
  document.body.style.overflow = 'auto'
  isEditing.value = '' 
}

function addNote() {
  if (!checkTextArea()) return 

  if (!isEditing.value) {
    notes.value.push({
      id: crypto.randomUUID(),
      text: newNote.value,
      date: new Date().toLocaleDateString('pt-BR'),
      color: `hsl(${Math.random() * 360}, 100%, 75%)`,
      isHovered: false
    })
  } else {
    const index = notes.value.findIndex(note => note.id === isEditing.value)

    notes.value[index].text = newNote.value
    notes.value[index].date = new Date().toLocaleDateString('pt-BR')
  }
  
  closeModal()
  
  localStorage.setItem('notes', JSON.stringify(notes.value))
}

function deleteNote(id) {
  notes.value.splice(
    notes.value.findIndex(note => note.id === id),
    1
  )
  localStorage.setItem('notes', JSON.stringify(notes.value))
}

function editNote({text, id}) {
  isEditing.value = id
  newNote.value = text
  
  openModal()
}
</script>

<template>
  <main>
    <transition name="overlay">
      <div v-if="showModal" @click="closeModal" class="overlay"></div>
    </transition>

    <transition name="modal">
      <div v-if="showModal" class="modal">
        <div class="modal-header">
          <p class="new-note">New note</p>

          <Icon @click="closeModal" class="close" icon="material-symbols:cancel" width="30px" color="#333" />
        </div>

        <textarea v-model.trim="newNote" @input="checkTextArea" :style="{ borderColor: textAreaBorderColor }"></textarea>
        
        <button @click="addNote" class="addNote">{{ isEditing ? 'Edit' : 'Add note' }}</button>
      </div>
    </transition>


    <header :class="{'header-shadow': hasScrolled}">
      <div class="spacer">
        <h1>Notes</h1>

        <Icon @click="openModal" class="newNote" icon="material-symbols:add-circle" width="60px" color="#333" />
      </div>
    </header>

    <div class="cards-container">
      <transition-group name="card">
        <div v-for="note in notes"
          class="card"
          @mouseover="note.isHovered = true"
          @mouseleave="note.isHovered = false" 
          :key="note.id" 
          :style="{ backgroundColor: note.color }">
  
          <p class="text">{{ note.text }}</p>

          <div class="card-bottom" :style="{backgroundColor: note.color}"> 
            <p class="date">{{ note.date }}</p>

            <div class="icons">
              <Icon v-if="note.isHovered" @click="editNote(note)" class="edit" icon="material-symbols:edit-square-rounded" color="#333" width="20px"/>

              <Icon v-if="note.isHovered" @click="deleteNote(note.id)" class="delete" icon="ic:outline-remove-circle" color="#333" width="20px"/>
            </div>

            
          </div>
        </div>
      </transition-group>
    </div>
  </main>
</template>

<style scoped>
header {
  background-color: #fff;
  width: 100%;
  position: sticky;
  top: 0;
  padding: 10px;
  margin-bottom: 50px;
  transition: box-shadow 0.3s;
  z-index: 2;
}

.spacer {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  width: 1000px;
  margin: 0 auto;
}

.header-shadow {
  box-shadow: 0px 10px 10px -10px #000;
}

h1 {
  font-size: 80px;
  color: #333;
  margin: 0;
}

.newNote {
  cursor: pointer;
  transition: transform 0.2s;
}

.newNote:hover {
  transform: scale(1.1);
}

.newNote:active {
  transform: scale(0.9);
}

.cards-container {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  max-width: 1000px;
  margin: 0 auto;
}

.card {
  width: 200px;
  height: 240px;
  border-radius: 15px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  margin-right: 20px;
  margin-bottom: 20px;
  overflow-y: auto;
  -ms-overflow-style: none;
  scrollbar-width: none;
  transition: 0.2s;
}

.card:hover {
  transform: scale(1.05);
}

.text, .date {
  margin: 12px;
  font-size: 20px;
  color: #333;
  font-weight: bold;
  word-wrap: break-word;
  overflow-wrap: break-word;
}

.date {
  font-size: 16px;
  font-weight: normal
}

.card-bottom {
  position: sticky;
  bottom: 0;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}

.icons {
  margin: 12px;
  gap: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.delete, .edit {
  cursor: pointer;
  transition: 0.2s;
}

.delete:hover, .edit:hover {
  transform: scale(1.3);
}

.delete:active, .edit:active {
  transform: scale(1);
}

.overlay {
  position: fixed;
  background-color: rgba(51, 51, 51, 0.75);
  width: 100vw;
  height: 100vh;
  z-index: 99;
  top: 0;
  left: 0;
}

.modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 600px;
  background-color: #fff;
  display: flex;
  flex-direction: column;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0px 0px 10px #000;
  z-index: 100; 
}

.modal-header {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}

.new-note {
  font-size: 20px;
  font-weight: bold;
}

textarea {
  width: 100%;
  height: 150px;
  margin-bottom: 30px;
  padding: 10px;
  border-radius: 10px;
  font-size: 18px;
  resize: none;
  border: solid 2px;
  outline: none;
  transition: border 0.1s;
}

.addNote {
  width: 100%;
  color: #fff;
  padding: 10px;
  background-color: #333;
  border-radius: 10px;
  font-size: 20px;
  border: none;
  cursor: pointer;
  transition: transform 0.2s;
}

.addNote:hover {
  transform: scale(1.02);
}

.addNote:active {
  transform: scale(0.98);
}

.close {
  margin-left: auto;
  margin-bottom: 10px;
  cursor: pointer;
  transition: transform 0.2s;
}

.close:hover {
  transform: scale(1.1);
}

.close:active {
  transform: scale(0.9);
}

@keyframes zoomIn {
  from {
    transform: translate(-50%, -50%) scale(0);
  }
  to {
    transform: translate(-50%, -50%) scale(1);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0%;
  }
  to {
    opacity: 100%;
  }
}

@keyframes slideIn {
  from {
    transform: translateY(20%);
  }
  to {
    transform: translateY(0);
  }
}

@keyframes slideOut {
  from {
    transform: translateY(0);
  }
  to {
    transform: translateY(-20%);
  }
}

.modal-enter-active {
  animation: zoomIn 0.3s forwards ease-in-out;
}

.modal-leave-active {
  animation: zoomIn 0.3s forwards reverse ease-in-out;
}

.overlay-enter-active {
  animation: fadeIn 0.2s forwards ease-in-out;
}

.overlay-leave-active {
  animation: fadeIn 0.2s forwards reverse ease-in-out;
}

.card-enter-active {
  animation: slideIn 0.3s forwards ease-out, fadeIn 0.2s forwards;
}

.card-leave-active {
  animation: slideOut 0.2s forwards, fadeIn 0.2s forwards reverse;
}
</style>