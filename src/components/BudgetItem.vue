<script lang="ts" setup>
import { ref } from 'vue';
import BudgetForm from './BudgetForm.vue';
import { BudgetItem } from '../type';

defineProps<{
  budget: BudgetItem;
}>();

const show = ref(false);

const editBudget = (budget: BudgetItem) => {
  console.log(budget);
};
</script>

<template>
  <div class="budget-item">
    <w-tag class="type">{{ budget.type }}</w-tag>
    <div class="price">{{ budget.price }}</div>
    <div class="desc">{{ budget.desc }}</div>
    <div class="date">{{ budget.created.split(' ')[0] }}</div>
    <w-button outline sm @click="show = true">🔧</w-button>
    <BudgetForm
      :show="show"
      :budget="budget"
      @cancel="show = false"
      @confirm="editBudget"
    />
  </div>
</template>

<style lang="scss" scoped>
.budget-item {
  border-bottom: 1px solid #e6e6e6;
}
</style>