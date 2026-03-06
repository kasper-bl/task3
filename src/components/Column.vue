<template>
  <div class="column">
    <h3>{{ column.title }}</h3>
    <button v-if="canCreate" @click="showModal = true">Добавить карточку</button>

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
            required
          />
          <small v-if="deadlineError" class="error">{{ deadlineError }}</small>
        </div>

        <div class="modal-actions">
          <button @click="closeModal">Отмена</button>
          <button class="btn-primary" @click="createCard">Создать</button>
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
import { defineProps, defineEmits, ref } from 'vue';
import Card from './Card.vue';

const props = defineProps({
  column: { type: Object, required: true },
  canCreate: { type: Boolean, default: false },
  canEdit: { type: Boolean, default: false },
  canDelete: { type: Boolean, default: false },
  boardData: { type: Object, required: true }
});

const emit = defineEmits(['moveCard', 'editCard', 'deleteCard']);

const showModal = ref(false);
const deadlineError = ref('');
const newCard = ref({
  title: '',
  description: '',
  deadlineLocal: '',
  _deadlineISO: null
});

function closeModal() {
  showModal.value = false;
  newCard.value = { title: '', description: '', deadlineLocal: '', _deadlineISO: null };
  deadlineError.value = '';
}

function onDateTimeChange(e) {
  const value = e.target.value;
  if (!value) {
    newCardISO = null;
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

function onMoveToColumn({ card, targetIndex }) {
  const fromIndex = props.column.id - 1;
  emit('moveCard', { card, fromColIndex: fromIndex, toColIndex: targetIndex });
}
</script>

<style scoped>
.column {
  border: 1px solid #ccc;
  padding: 10px;
  width: 300px;
  background-color: #f9f9f9;
  min-height: 500px;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal-content {
  background: white;
  padding: 20px;
  border-radius: 8px;
  width: 90%;
  max-width: 400px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.2);
}

.form-group {
  margin-bottom: 16px;
}

.form-group label {
  display: block;
  margin-bottom: 6px;
  font-weight: 500;
}

.form-group input,
.form-group textarea {
  width: 370px;
  padding: 8px 12px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
}

.form-group textarea {
  resize: vertical;
}

.error {
  color: #d32f2f;
  font-size: 12px;
  display: block;
  margin-top: 4px;
}

.modal-actions {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}

.modal-actions button {
  flex: 1;
  padding: 8px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.modal-actions button:first-child {
  background: #f1f1f1;
  color: #333;
}

.btn-primary {
  background: #4caf50;
  color: white;
}

.btn-primary:hover {
  background: #45a049;
}
</style>