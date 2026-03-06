<template>
  <div class="board">
    <Column
      :column="store.columns[0]"
      :can-create="true"
      :can-edit="true"
      :can-delete="true"
      @move-card="handleMoveCard"
      @edit-card="openEditModal"
      @delete-card="confirmDelete"
    />
    <Column
      :column="store.columns[1]"
      :can-create="false"
      :can-edit="true"
      :can-delete="false"
      @move-card="handleMoveCard"
      @edit-card="openEditModal"
    />
    <Column
      :column="store.columns[2]"
      :can-create="false"
      :can-edit="true"
      :can-delete="false"
      @move-card="handleMoveCard"
      @edit-card="openEditModal"
    />
    <Column
      :column="store.columns[3]"
      :can-create="false"
      :can-edit="false"
      :can-delete="false"
    />
  </div>
</template>

<script setup>
import { watch } from 'vue';
import Column from './Column.vue';
import { store, saveState } from '../store.js';

function handleMoveCard({ card, fromColIndex, toColIndex }) {
  const fromCol = store.columns[fromColIndex];
  const toCol = store.columns[toColIndex];

  fromCol.cards = fromCol.cards.filter(c => c.id !== card.id);
  toCol.cards.push(card);

  if (toColIndex === 3) {
    const deadline = new Date(card.deadline);
    const now = new Date();
    card.status = now > deadline ? 'Просрочена' : 'Выполнена в срок';
  }

  saveState();
}

function openEditModal(card) {
  alert(`Редактирование карточки: ${card.title}`);
}

function confirmDelete(cardId) {
  if (confirm("Удалить карточку?")) {
    const col = store.columns[0]; 
    col.cards = col.cards.filter(c => c.id !== cardId);
    saveState();
  }
}

watch(() => store, () => {
  saveState();
}, { deep: true });
</script>

<style>
.board {
  display: flex;
  gap: 20px;
}
</style>