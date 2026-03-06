<template>
  <div class="card">
    <h4>{{ card.title }}</h4>
    <p><strong>Создана:</strong> {{ formatDate(card.createdAt) }}</p>
    <p><strong>Дедлайн:</strong> {{ formatDate(card.deadline) }}</p>
    <p><strong>Описание:</strong> {{ card.description }}</p>
    <p v-if="card.lastEditedAt"><strong>Редактировалась:</strong> {{ formatDate(card.lastEditedAt) }}</p>
    <p v-if="card.returnReason"><strong>Причина возврата:</strong> {{ card.returnReason }}</p>
    <p v-if="card.status"><strong>Статус:</strong> {{ card.status }}</p>

    <div class="actions">
      <button v-if="canEdit" @click="editCard">✎ Ред.</button>
      <button v-if="canDelete" @click="$emit('delete')">🗑 Удалить</button>
      <button v-if="columnIndex === 0" @click="moveTo(1)">→ В работу</button>
      <button v-if="columnIndex === 1" @click="moveTo(2)">→ Тестирование</button>
      <button v-if="columnIndex === 2" @click="showReturnOrComplete">→ Выполнено</button>
    </div>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue';

const props = defineProps({
  card: { type: Object, required: true },
  canEdit: { type: Boolean, default: false },
  canDelete: { type: Boolean, default: false },
  columnIndex: { type: Number, required: true }
});

const emit = defineEmits(['edit', 'delete', 'moveToColumn']);

function formatDate(dateStr) { 
  return new Date(dateStr).toLocaleString('ru-RU');
}

function editCard() {
  props.card.lastEditedAt = new Date().toISOString();
  alert('Редактирование (заглушка)');
}

function moveTo(targetIndex) {
  emit('moveToColumn', { card: props.card, targetIndex });
}

function showReturnOrComplete() {
  const reason = prompt('Причина возврата (оставьте пустым для завершения):');
  if (reason !== null) {
    if (reason.trim()) {
      props.card.returnReason = reason;
      emit('moveToColumn', { card: props.card, targetIndex: 1 });
    } else {
      emit('moveToColumn', { card: props.card, targetIndex: 3 });
    }
  }
}
</script>

<style scoped>
.card {
  border: 1px solid #ddd;
  margin-bottom: 10px;
  padding: 10px;
  background: white;
  box-shadow: 0 1px 3px rgba(0,0,0,0.1);
}
.actions { margin-top: 8px; }
button {
  margin: 2px 4px;
  padding: 3px 6px;
  font-size: 12px;
  cursor: pointer;
}
</style>