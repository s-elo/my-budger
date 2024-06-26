<script lang="ts" setup>
import { computed, ref, watch } from 'vue';
import { BudgetItem, BudgetType } from '../type';
import { useBudgetStore } from '../store';
import { QForm } from 'quasar';
import DatePicker from './DatePicker.vue';

const budgetStore = useBudgetStore();
const props = defineProps<{
  budget?: BudgetItem;
  show: boolean;
  loading?: boolean;
  mode?: 'Add' | 'Edit';
}>();

const emits = defineEmits(['cancel', 'confirm']);

const budgetFormRef = ref<QForm | null>(null);
const formData = ref({
  price: String(props.budget?.price || ''),
  tags: props.budget?.tags || [],
  desc: props.budget?.desc || '',
  type: props.budget?.type || BudgetType.spending,
  created: props.budget?.created || Date.now(),
});
const filterTag = ref<string>('');

watch(
  () => props.show,
  () => {
    if (props.show) {
      formData.value = {
        price: String(props.budget?.price || ''),
        tags: props.budget?.tags || [],
        desc: props.budget?.desc || '',
        type: props.budget?.type || BudgetType.spending,
        created: props.budget?.created || Date.now(),
      };
    } else {
      formData.value = {
        price: '',
        tags: [],
        desc: '',
        type: BudgetType.spending,
        created: Date.now(),
      };
    }
  },
);

const tagOptions = computed(() =>
  Object.keys({ ...budgetStore.allTags }).filter((tag) => {
    if (filterTag.value) {
      return tag.includes(filterTag.value);
    }

    return true;
  }),
);
const calculatedPrice = computed(() => {
  if (/[\+\*\-\/]/.test(formData.value.price)) {
    try {
      return eval(formData.value.price) as string;
    } catch {
      return 0;
    }
  }

  return formData.value.price || 0;
});

const confirm = async () => {
  const valid = await (budgetFormRef.value as QForm)?.validate();

  console.log(valid);
  console.log(formData.value);
  if (!valid) return;

  emits(
    'confirm',
    {
      ...(props.budget || {}),
      ...formData.value,
      price: Number(String(Number(calculatedPrice.value).toFixed(2))),
    },
    props.budget,
  );
};

const cancel = () => {
  emits('cancel');
};
</script>

<template>
  <q-dialog
    :model-value="show"
    persistent
    :full-width="true"
    position="bottom"
    transition-show="slide-up"
    transition-hide="slide-down"
  >
    <q-card class="bg-white form-container">
      <q-icon name="close" class="back-icon" @click="cancel" />
      <q-form ref="budgetFormRef" class="form">
        <div class="header">
          <q-btn-toggle
            v-model="formData.type"
            toggle-color="primary"
            class="btn-toggle"
            size="xs"
            dense
            no-caps
            :options="[
              { label: 'spending', value: 'spending' },
              { label: 'income', value: 'income' },
            ]"
          />
          <DatePicker v-model="formData.created" class="date-picker" />
        </div>
        <q-input
          v-model="formData.price"
          lazy-rule
          :rules="[
            (val) =>
              (!!val && /^[\d\.\+\-\*\/]+$/.test(val)) || 'Invalid Expression.',
          ]"
        >
          <template #prepend>
            <div class="dollar-con">¥ {{ calculatedPrice || '' }}</div>
          </template>
        </q-input>
        <q-select
          v-model="formData.tags"
          placeholder="Tags"
          use-input
          use-chips
          multiple
          input-debounce="0"
          new-value-mode="add"
          :options="tagOptions"
          lazy-rule
          :rules="[(val) => val.length || 'At least one tag.']"
          @filter="(val, update) => update(() => (filterTag = val))"
        />
        <q-input v-model="formData.desc" autogrow placeholder="Any Note?" />
        <q-btn
          :loading="loading"
          class="confirm-btn"
          color="primary"
          no-caps
          @click="confirm"
        >
          OK
          <template #loading>
            <q-spinner-gears class="on-left" />
          </template>
        </q-btn>
      </q-form>
    </q-card>
  </q-dialog>
</template>

<style lang="scss" scoped>
.form-container {
  padding: 16px;
  .form {
    display: flex;
    flex-direction: column;
  }
  .back-icon {
    cursor: pointer;
    margin-bottom: 7px;
  }
  .btn-toggle {
    margin-top: 16px;
  }
  .confirm-btn {
    align-self: self-end;
    margin-top: 16px;
  }
  .header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    .date-picker {
      margin-top: 16px;
    }
  }
}
</style>
