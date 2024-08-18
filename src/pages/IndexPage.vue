<template>
  <q-layout>
    <q-page-container>
      <q-page padding>
        <div class="row">
          <div class="col-12 col-md-4">
            <q-form ref="formRef">
              <q-input
                v-model="inputColumn"
                :rules="[required]"
                filled
                label="Columns"
                @update:model-value="validateAndGenerate"
              >
                <template v-slot:hint>
                  Enter column names separated by comma
                </template>
              </q-input>
              <q-btn label="Copy" @click="copyToClipboards" />
            </q-form>
          </div>
          <div class="col-12 row">
            <div class="col-12">
              <pre>{{ formatColumns() }}</pre>
            </div>
            <div class="col-12 col-md-8">
              <q-card>
                <q-card-section> Generated Table </q-card-section>
                <q-card-section>
                  <q-table :columns="finalColumns as any" :rows="[]" />
                </q-card-section>
              </q-card>
            </div>
          </div>
        </div>
      </q-page>
    </q-page-container>
  </q-layout>
</template>

<script lang="ts" setup>
import { Ref, ref } from 'vue';
import { copyToClipboard, Notify, QForm } from 'quasar';
const required = (val: string): true | string => {
  if (!val || val.length === 0) {
    return 'This field is required';
  }

  return true;
};
const inputColumn = ref('');
const formRef = ref() as Ref<QForm | undefined>;

type Column = {
  name: string;
  label: string;
  align: string;
  field: string;
  sortable?: boolean;
  format?: string;
};

const finalColumns = ref<Column[]>([
  {
    name: 'index',
    label: 'S.N.',
    align: 'left',
    field: 'index',
    format: "(val: number) => `${val || ''}`",
  },
  {
    name: 'actions',
    label: 'Action',
    align: 'left',
    field: 'actions',
    sortable: false,
  },
]);

const validateAndGenerate = () => {
  formRef.value
    ?.validate()
    .then(() => {
      generateColumns();
    })
    .catch(() => {
      Notify.create({
        message: 'Please fill the form',
        color: 'negative',
        position: 'top-right',
        timeout: 2000,
      });
    });
};

const generateColumns = () => {
  const fieldArray = inputColumn.value.split(',').map((field) => field.trim());
  const newColumns = fieldArray.map((field) => ({
    name: field,
    label: upperCaseWord(field.replace('_', ' ')),
    align: 'left',
    field: field,
    sortable: true,
  })) as Column[];

  const indexColumn = finalColumns.value.find((col) => col.name === 'index');
  const actionsColumn = finalColumns.value.find(
    (col) => col.name === 'actions',
  );

  finalColumns.value = [indexColumn, ...newColumns, actionsColumn].filter(
    Boolean,
  ) as Column[];
};

const upperCaseWord = (str: string) => {
  return str.replace(/_/g, ' ').replace(/\b\w/g, (char) => char.toUpperCase());
};

const formatColumns = () => {
  return JSON.stringify(finalColumns.value, null, 2)
    .replace(/"(\w+)":/g, '$1:')
    .replace(/"/g, '')
    .replace(/\\n/g, '\n')
    .replace(/\\t/g, '\t');
};

const copyToClipboards = async () => {
  try {
    await copyToClipboard(formatColumns()).then(() => {
      Notify.create({
        message: 'Columns copied to clipboard',
        color: 'positive',
        position: 'center',
        timeout: 2000,
      });
    });
  } catch (err) {
    Notify.create({
      message: 'Failed to copy columns to clipboard',
      color: 'negative',
      position: 'top-right',
      timeout: 2000,
    });
  }
};
</script>
