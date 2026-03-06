<template>
  <div class="board">
    <Column
      :column="columns[0]"
      :can-create="true"
      :can-edit="true"
      :can-delete="true"
      :board-data="{ columns, nextCardId }"
      @move-card="handleMoveCard"
      @edit-card="openEditModal"
      @delete-card="confirmDelete"
    />
    <Column
      :column="columns[1]"
      :can-create="false"
      :can-edit="true"
      :can-delete="false"
      :board-data="{ columns, nextCardId }"
      @move-card="handleMoveCard"
      @edit-card="openEditModal"
    />
    <Column
      :column="columns[2]"
      :can-create="false"
      :can-edit="true"
      :can-delete="false"
      :board-data="{ columns, nextCardId }"
      @move-card="handleMoveCard"
      @edit-card="openEditModal"
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

const nextCardId = ref(1);

// Загрузка из localStorage
watchEffect(() => {
  const saved = localStorage.getItem('kanbanStateV2');
  if (saved) {
    try {
      const parsed = JSON.parse(saved);
      columns.value = parsed.columns || columns.value;
      nextCardId.value = parsed.nextCardId || 1;
    } catch (e) {
      console.warn('Ошибка загрузки состояния', e);
    }
  }
});

// Сохранение
watchEffect(() => {
  localStorage.setItem('kanbanStateV2', JSON.stringify({ 
    columns: columns.value, 
    nextCardId: nextCardId.value 
  }));
});

function handleMoveCard({ card, fromColIndex, toColIndex }) {
  const fromCol = columns.value[fromColIndex];
  const toCol = columns.value[toColIndex];

  fromCol.cards = fromCol.cards.filter(c => c.id !== card.id);
  toCol.cards.push(card);

  // Проверка дедлайна при переходе в 4-й столбец
  if (toColIndex === 3) {
    const deadline = new Date(card.deadline);
    const now = new Date();
    card.status = now > deadline ? 'Просрочена' : 'Выполнена в срок';
  }
}

function openEditModal(card) {
  alert(`Редактирование: ${card.title}`);
}

function confirmDelete(cardId) {
  if (confirm('Удалить карточку?')) {
    // Ищем карточку во всех столбцах
    for (let i = 0; i < columns.value.length; i++) {
      const index = columns.value[i].cards.findIndex(c => c.id === cardId);
      if (index !== -1) {
        columns.value[i].cards.splice(index, 1);
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
}
</style>