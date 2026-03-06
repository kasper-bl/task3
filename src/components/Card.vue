<template>
  <div class="card">
    <h4>{{ card.title }}</h4>
    <p><strong>Дата создания:</strong> {{ formatDate(card.createdAt) }}</p>
    <p><strong>Дедлайн:</strong> {{ formatDate(card.deadline) }}</p>
    <p><strong>Описание:</strong> {{ card.description }}</p>
    <p v-if="card.lastEditedAt"><strong>Последнее редактирование:</strong> {{ formatDate(card.lastEditedAt) }}</p>
    <p v-if="card.returnReason"><strong>Причина возврата:</strong> {{ card.returnReason }}</p>
    <p v-if="card.status"><strong>Статус:</strong> {{ card.status }}</p>

    <div class="actions">
      <button v-if="canEdit" @click="editCard">Редактировать</button>
      <button v-if="canDelete" @click="$emit('delete')">Удалить</button>

      <!-- Перемещение -->
      <button v-if="columnIndex === 0" @click="moveTo(1)">→ В работу</button>
      <button v-if="columnIndex === 1" @click="moveTo(2)">→ Тестирование</button>
      <button v-if="columnIndex === 2" @click="showReturnOrComplete">→ Выполненные</button>
    </div>
  </div>
</template>

<script setup>
import { inject } from 'vue';

const props = defineProps(['card', 'canEdit', 'canDelete']);
defineEmits(['edit', 'delete', 'moveToColumn']);

// Для определения текущего столбца
const columnIndex = inject('columnIndex');

function formatDate(dateStr) {
  return new Date(dateStr).toLocaleString();
}

function editCard() {
  props.card.lastEditedAt = new Date().toISOString();
  // Здесь можно открыть модальное окно, пока просто alert
  alert('Открыто окно редактирования');
}

function moveTo(targetIndex) {
  props.$emit('moveToColumn', { card: props.card, targetIndex });
}

function showReturnOrComplete() {
  const reason = prompt('Хотите вернуть карточку? Введите причину (или оставьте пустым для завершения):');
  if (reason !== null) {
    if (reason.trim()) {
      props.card.returnReason = reason;
      props.$emit('moveToColumn', { card: props.card, targetIndex: 1 }); // вернуть во 2-й
    } else {
      props.$emit('moveToColumn', { card: props.card, targetIndex: 3 }); // в 4-й
    }
  }
}
</script>

<style scoped>
.card {
  border: 1px solid #ddd;
  margin-bottom: 10px;
  padding: 10px;
  background-color: white;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
.actions {
  margin-top: 10px;
}
button {
  margin-right: 5px;
  margin-bottom: 5px;
}
</style>