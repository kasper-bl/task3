<template>
  <div class="board">
    <Column
      :column="columns[0]"
      :can-create="true"
      :can-edit="true"
      :can-delete="true"
      :board-data="{ columns, nextCardId }"
      :editing-card="editingCard"
      @move-card="handleMoveCard"
      @edit-card="startEditCard"
      @delete-card="confirmDelete"
      @close-edit-modal="closeEditModal"
    />
    <Column
      :column="columns[1]"
      :can-create="false"
      :can-edit="true"
      :can-delete="false"
      :board-data="{ columns, nextCardId }"
      :editing-card="editingCard"
      @move-card="handleMoveCard"
      @edit-card="startEditCard"
      @close-edit-modal="closeEditModal"
    />
    <Column
      :column="columns[2]"
      :can-create="false"
      :can-edit="true"
      :can-delete="false"
      :board-data="{ columns, nextCardId }"
      :editing-card="editingCard"
      @move-card="handleMoveCard"
      @edit-card="startEditCard"
      @close-edit-modal="closeEditModal"
    />
    <Column
      :column="columns[3]"
      :can-create="false"
      :can-edit="false"
      :can-delete="false"
      :board-data="{ columns, nextCardId }"
    />
  </div>
</template>

<script setup>
import { ref, watchEffect } from 'vue';
import Column from './Column.vue';

const columns = ref([
  { id: 1, title: 'Запланированные задачи', cards: [] },
  { id: 2, title: 'Задачи в работе', cards: [] },
  { id: 3, title: 'Тестирование', cards: [] },
  { id: 4, title: 'Выполненные задачи', cards: [] }
]);

let nextCardId = 1;
const editingCard = ref(null);

watchEffect(() => {
  const saved = localStorage.getItem('kanbanStateV2');
  if (saved) {
    try {
      const parsed = JSON.parse(saved);
      columns.value = parsed.columns || columns.value;
      nextCardId = parsed.nextCardId || 1;
    } catch {}
  }
});

watchEffect(() => {
  localStorage.setItem('kanbanStateV2', JSON.stringify({ columns: columns.value, nextCardId }));
});

function handleMoveCard({ card, fromColIndex, toColIndex }) {
  const fromCol = columns.value[fromColIndex];
  const toCol = columns.value[toColIndex];
  fromCol.cards = fromCol.cards.filter(c => c.id !== card.id);
  toCol.cards.push(card);

  if (toColIndex === 3) {
    const deadline = new Date(card.deadline);
    const now = new Date();
    card.status = now > deadline ? 'Просрочена' : 'Выполнена в срок';
  }
}

function startEditCard(card) {
  editingCard.value = card;
}

function closeEditModal() {
  editingCard.value = null;
}

function confirmDelete(cardId) {
  if (confirm('Удалить?')) {
    for (let i = 0; i < columns.value.length; i++) {
      const idx = columns.value[i].cards.findIndex(c => c.id === cardId);
      if (idx !== -1) {
        columns.value[i].cards.splice(idx, 1);
        break;
      }
    }
  }
}
</script>

<style>
.board {
  display: flex;
  gap: 20px;
  padding: 10px;
  background: linear-gradient(135deg, #e8f5e9 0%, #c8e6c9 100%);
  min-height: 100vh;
  padding: 20px;
}
</style>