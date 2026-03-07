<template>
  <div class="column">
    <h3>{{ column.title }}</h3>
    <button v-if="canCreate" @click="showModal = true" class="btn-add">Добавить карточку</button>

    <div v-if="showModal" class="modal-overlay" @click.self="closeModal">
      <div class="modal-content" @click.stop>
        <h4>Создать новую задачу</h4>
        <div class="form-group">
          <label>Название:</label>
          <input v-model="newCard.title" type="text" placeholder="Введите название" required />
        </div>
        <div class="form-group">
          <label>Описание:</label>
          <textarea v-model="newCard.description" rows="3" placeholder="Описание задачи"></textarea>
        </div>
        <div class="form-group">
          <label>Дедлайн:</label>
          <input
            v-model="newCard.deadlineLocal"
            type="datetime-local"
            @change="onDateTimeChange"
          />
          <small v-if="deadlineError" class="error">{{ deadlineError }}</small>
        </div>

        <div class="modal-actions">
          <button @click="closeModal" class="btn-cancel">Отмена</button>
          <button class="btn-primary" @click="createCard">Создать</button>
        </div>
      </div>
    </div>

    <div v-if="editingCard && editingCard.id === cardBeingEdited?.id" class="modal-overlay" @click.self="closeEditModal">
      <div class="modal-content" @click.stop>
        <h4>Редактировать задачу</h4>
        <div class="form-group">
          <label>Название:</label>
          <input v-model="editCardData.title" type="text" required />
        </div>
        <div class="form-group">
          <label>Описание:</label>
          <textarea v-model="editCardData.description" rows="3"></textarea>
        </div>
        <div class="form-group">
          <label>Дедлайн:</label>
          <input
            v-model="editCardData.deadlineLocal"
            type="datetime-local"
            @change="onEditDateTimeChange"
            required
          />
        </div>

        <div class="modal-actions">
          <button @click="closeEditModal" class="btn-cancel">Отмена</button>
          <button class="btn-primary" @click="saveEditCard">Сохранить</button>
        </div>
      </div>
    </div>

    <div class="cards">
      <Card
        v-for="card in column.cards"
        :key="card.id"
        :card="card"
        :can-edit="canEdit"
        :can-delete="canDelete"
        :column-index="column.id - 1"
        @move-to-column="onMoveToColumn"
        @edit="$emit('editCard', card)"
        @delete="$emit('deleteCard', card.id)"
      />
    </div>
  </div>
</template>

<script setup>
import { defineProps, defineEmits, ref, watch } from 'vue';
import Card from './Card.vue';

const props = defineProps({
  column: { type: Object, required: true },
  canCreate: { type: Boolean, default: false },
  canEdit: { type: Boolean, default: false },
  canDelete: { type: Boolean, default: false },
  boardData: { type: Object, required: true },
  editingCard: { type: Object }
});

const emit = defineEmits(['moveCard', 'editCard', 'deleteCard', 'closeEditModal']);

const showModal = ref(false);
const deadlineError = ref('');
const newCard = ref({
  title: '',
  description: '',
  deadlineLocal: '',
  _deadlineISO: null
});

const cardBeingEdited = ref(null);
const editCardData = ref({
  title: '',
  description: '',
  deadlineLocal: ''
});

watch(() => props.editingCard, (newVal) => {
  if (newVal) {
    cardBeingEdited.value = newVal;
    editCardData.value = {
      title: newVal.title,
      description: newVal.description,
      deadlineLocal: newVal.deadline ? new Date(newVal.deadline).toISOString().slice(0, 16) : ''
    };
  }
});

function closeModal() {
  showModal.value = false;
  newCard.value = { title: '', description: '', deadlineLocal: '',deadlineISO: null };
  deadlineError.value = '';
}

function onDateTimeChange(e) {
  const value = e.target.value;
  if (!value) {
    newCard.value._deadlineISO = null;
    deadlineError.value = 'Дедлайн обязателен';
    return;
  }

  const date = new Date(value);
  if (isNaN(date.getTime())) {
    newCard.value._deadlineISO = null;
    deadlineError.value = 'Неверная дата или время';
    return;
  }

  newCard.value._deadlineISO = date.toISOString();
  deadlineError.value = '';
}

function createCard() {
  const { title, description, _deadlineISO } = newCard.value;

  if (!title.trim()) {
    alert('Название обязательно!');
    return;
  }
  if (!_deadlineISO) {
    alert('Выберите дедлайн');
    return;
  }

  const now = new Date().toISOString();
  const card = {
    id: props.boardData.nextCardId++,
    createdAt: now,
    title,
    description,
    deadline: _deadlineISO,
    lastEditedAt: now,
    status: null,
    returnReason: null
  };

  props.column.cards.push(card);
  closeModal();
}

function onEditDateTimeChange(e) {
  const value = e.target.value;
  if (value) {
    const date = new Date(value);
    if (!isNaN(date.getTime())) {
      editCardData.value._deadlineISO = date.toISOString();
    }
  }
}

function saveEditCard() {
  if (!editCardData.value.title.trim()) {
    alert('Название обязательно!');
    return;
  }

  cardBeingEdited.value.title = editCardData.value.title;
  cardBeingEdited.value.description = editCardData.value.description;
  if (editCardData.value._deadlineISO) {
    cardBeingEdited.value.deadline = editCardData.value._deadlineISO;
  }
  cardBeingEdited.value.lastEditedAt = new Date().toISOString();

  emit('closeEditModal');
}

function closeEditModal() {
  emit('closeEditModal');
}

function onMoveToColumn({ card, targetIndex }) {
  const fromIndex = props.column.id - 1;
  emit('moveCard', { card, fromColIndex: fromIndex, toColIndex: targetIndex });
}
</script>

<style scoped>
.column {
  border: 2px solid #a5d6a7;
  padding: 15px;
  width: 320px;
  min-height: 500px;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(76, 175, 80, 0.15);
}

.column h3 {
  color: #000000;
  margin-top: 0;
  margin-bottom: 15px;
  text-align: center;
  font-weight: 600;
}

.btn-add {
  background: #4caf50;
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 6px;
  cursor: pointer;
  width: 100%;
  font-weight: 500;
  margin-bottom: 15px;
  transition: background 0.2s;
}

.btn-add:hover {
  background: #43a047;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 10
}

.modal-content {
  background: white;
  padding: 25px;
  border-radius: 10px;
  width: 90%;
  max-width: 420px;
  box-shadow: 0 8px 24px rgba(76, 175, 80, 0.3);
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  margin-bottom: 8px;
  font-weight: 600;
  color: #000000;
}

.form-group input,
.form-group textarea {
  width: 94%;
  padding: 10px 12px;
  border: 2px solid #c8e6c9;
  border-radius: 6px;
  font-size: 14px;
  transition: border 0.2s;
}

.form-group input:focus,
.form-group textarea:focus {
  outline: none;
  border-color: #4caf50;
}

.form-group textarea {
  resize: vertical;
}

.error {
  color: #c62828;
  font-size: 12px;
  display: block;
  margin-top: 4px;
}

.modal-actions {
  display: flex;
  gap: 12px;
  margin-top: 20px;
}

.modal-actions button {
  flex: 1;
  padding: 10px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: background 0.2s;
}

.btn-cancel {
  background: #f1f8e9;
  color: #558b2f;
}

.btn-cancel:hover {
  background: #dcedc8;
}

.btn-primary {
  background: #4caf50;
  color: white;
}

.btn-primary:hover {
  background: #43a047;
}
</style>