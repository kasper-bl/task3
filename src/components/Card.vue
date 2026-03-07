<template>
  <div class="card" :class="deadlineClass">
    <h4>{{ card.title }}</h4>
    <p><strong>Создана:</strong> {{ formatDate(card.createdAt) }}</p>
    <p><strong>Дедлайн:</strong> {{ formatDate(card.deadline) }}</p>
    <p><strong>Описание:</strong> {{ card.description }}</p>
    <p v-if="card.lastEditedAt"><strong>Редактировалась:</strong> {{ formatDate(card.lastEditedAt) }}</p>
    <p v-if="card.returnReason"><strong>Причина возврата:</strong> {{ card.returnReason }}</p>
    <p v-if="card.status"><strong>Статус:</strong> {{ card.status }}</p>

    <div class="actions">
      <button v-if="canEdit" @click="editCard" class="btn-edit">Редактировать</button>
      <button v-if="canDelete" @click="$emit('delete')" class="btn-delete">Удалить</button>

      <button v-if="columnIndex === 0" @click="moveTo(1)" class="btn-move">Переместить в работу</button>

      <button v-if="columnIndex === 1" @click="moveTo(2)" class="btn-move">Переместить в тестирование</button>

      <button
        v-if="columnIndex === 2"
        class="btn-complete"
        @click="completeTask"
      >
        Переместить в выполненные
      </button>
      <button
        v-if="columnIndex === 2"
        class="btn-return"
        @click="returnToWork"
      >
        Вернуть в работу
      </button>
    </div>
  </div>
</template>

<script setup>
import { defineProps, defineEmits, computed } from 'vue';

const props = defineProps({
  card: { type: Object, required: true },
  canEdit: { type: Boolean, default: false },
  canDelete: { type: Boolean, default: false },
  columnIndex: { type: Number, required: true }
});

const emit = defineEmits(['edit', 'delete', 'moveToColumn']);

const deadlineClass = computed(() => {
  if (!props.card.deadline) return '';

  const deadline = new Date(props.card.deadline);
  const now = new Date();
  const diffMs = deadline - now;
  const diffHours = diffMs / (1000 * 60 * 60);

  if (diffHours <= 24 && diffHours > 0) {
    return 'blink-red';
  } else if (diffHours <= 72 && diffHours > 0) {
    return 'blink-orange';
  }
  return '';
});

function formatDate(dateStr) {
  return new Date(dateStr).toLocaleString('ru-RU');
}

function editCard() {
  props.card.lastEditedAt = new Date().toISOString();
  emit('edit'); 
}

function moveTo(targetIndex) {
  emit('moveToColumn', { card: props.card, targetIndex });
}

function completeTask() {
  emit('moveToColumn', { card: props.card, targetIndex: 3 });
}

function returnToWork() {
  const reason = prompt('Причина возврата (обязательно)');
  if (reason !== null && reason.trim()) {
    props.card.returnReason = reason;
    emit('moveToColumn', { card: props.card, targetIndex: 1 });
  } else if (reason !== null) {
    alert('Укажите причину возврата.');
  }
}
</script>

<style scoped>
.card {
  border: 1px solid #c8e6c9;
  margin-bottom: 10px;
  padding: 15px;
  background: #ffffff;
  box-shadow: 0 2px 8px rgba(76, 175, 80, 0.2);
  border-radius: 8px;
  transition: box-shadow 0.3s ease;
}

.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(76, 175, 80, 0.3);
}

.blink-red {
  animation: pulseRed 2s ease-in-out infinite;
}

.blink-orange {
  animation: pulseOrange 2s ease-in-out infinite;
}

@keyframes pulseRed {
  0%, 100% { box-shadow: 0 0 5px 2px transparent; }
  50% { box-shadow: 0 0 12px 4px #ff0000; }
}

@keyframes pulseOrange {
  0%, 100% { box-shadow: 0 0 5px 2px transparent; }
  50% { box-shadow: 0 0 10px 3px #ff9900; }
}

.actions {
  margin-top: 12px;
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}

.actions button {
  margin: 2px 0;
  padding: 6px 12px;
  font-size: 12px;
  cursor: pointer;
  border: none;
  border-radius: 4px;
  transition: background 0.2s;
  text-align: center;
  min-width: 120px;
}

.btn-edit {
  background: #81c784;
  color: white;
}

.btn-edit:hover {
  background: #66bb6a;
}

.btn-delete {
  background: #e57373;
  color: white;
}

.btn-delete:hover {
  background: #ef5350;
}

.btn-move {
  background: #4caf50;
}

.btn-move:hover {
  background: #9ccc65;
}

.btn-complete {
  background: #4caf50;
  color: white;
}

.btn-complete:hover {
  background: #43a047;
}

.btn-return {
  background: #fbc02d;
  color: white;
}

.btn-return:hover {
  background: #f9a825;
}
</style>