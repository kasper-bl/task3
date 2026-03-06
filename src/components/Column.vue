<template>
  <div class="column">
    <h3>{{ column.title }}</h3>
    <button v-if="canCreate" @click="addCard">Добавить карточку</button>
    <div class="cards">
      <Card
        v-for="card in column.cards"
        :key="card.id"
        :card="card"
        :can-edit="canEdit"
        :can-delete="canDelete"
        @move-to-column="onMoveToColumn"
        @edit="emit('editCard', card)"
        @delete="emit('deleteCard', card.id)"
      />
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import Card from './Card.vue';
import { store } from '../store.js';

const props = defineProps(['column', 'canCreate', 'canEdit', 'canDelete']);
const emit = defineEmits(['moveCard', 'editCard', 'deleteCard']);

function addCard() {
  const now = new Date().toISOString();
  const newCard = {
    id: store.nextCardId++,
    createdAt: now,
    title: `Новая задача ${store.nextCardId - 1}`,
    description: 'Описание задачи',
    deadline: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000).toISOString(), // +7 дней
    lastEditedAt: now,
    status: null,
    returnReason: null
  };

  props.column.cards.push(newCard);
}

function onMoveToColumn({ card, targetIndex }) {
  emit('moveCard', { card, fromColIndex: props.column.id - 1, toColIndex: targetIndex });
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
</style>