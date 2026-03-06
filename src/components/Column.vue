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
        @edit="$emit('editCard', card)"
        @delete="$emit('deleteCard', card.id)"
      />
    </div>
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue';
import Card from './Card.vue';

const props = defineProps({
  column: { type: Object, required: true },
  canCreate: { type: Boolean, default: false },
  canEdit: { type: Boolean, default: false },
  can: { type: Boolean, default: false },
  boardData: { type: Object, required: true } 
});

const emit = defineEmits(['moveCard', 'editCard', 'deleteCard']);

function addCard() {
  const now = new Date().toISOString();
  const newCard = {
    id: props.boardData.nextCardId.value++,
    createdAt: now,
    title: `Новая задача ${props.boardData.nextCardId.value - 1}`,
    description: 'Описание задачи',
    deadline: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000).toISOString(),
    lastEditedAt: now,
    status: null,
    returnReason: null
  };

  props.column.cards.push(newCard);
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
</style>