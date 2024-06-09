<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" :rules="nameRules" />
        <q-input v-model="tempData.age" label="年齡" :rules="ageRules" />
        <q-btn
          v-if="mode === 'create'"
          color="primary"
          class="q-mt-md"
          @click="handleCreateUser"
          >新增</q-btn
        >
        <q-btn v-else color="primary" class="q-mt-md" @click="handleEditUser"
          >編輯</q-btn
        >
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<script setup lang="ts">
import { api } from 'boot/axios';
import { QTableProps, useQuasar } from 'quasar';
import { onMounted, ref } from 'vue';

interface btnType {
  label: string;
  icon: string;
  status: string;
}

interface user {
  id?: string;
  name: string;
  age: number;
}
const $q = useQuasar();
const mode = ref('create');
const blockData = ref([]);
const nameRules = ref([(val: string) => (val && val.length > 0) || '請勿空白']);
const ageRules = ref([
  (val: string) => /^[1-9]\d*$/.test(val) || '請輸入正整數',
  (val: string) => (val !== null && val !== '') || '請勿空白',
]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const tempData = ref({
  id: '',
  name: '',
  age: '',
});

async function loadData() {
  const res = await api.get('api/CRUDTest/a');
  blockData.value = res.data;
  mode.value = 'create';
  tempData.value = {
    id: '',
    name: '',
    age: '',
  };
}

async function handleCreateUser() {
  await api.post('api/CRUDTest', tempData.value);
  loadData();
}

function editUser(data: user) {
  console.log(data);
  tempData.value = {
    id: data.id || '',
    name: data.name,
    age: data.age.toString(),
  };
  mode.value = 'edit';
}

function deleteUser(id: string) {
  $q.dialog({
    title: '提示',
    message: '是否確定刪除該資料?',
    ok: {
      push: true,
    },
    cancel: {
      push: true,
      color: 'negative',
    },
    persistent: true,
  })
    .onOk(() => {
      handleDeleteUser(id);
    })
    .onCancel(() => {
      // console.log('>>>> Cancel')
    });
}

async function handleEditUser() {
  await api.patch('api/CRUDTest', tempData.value);
  loadData();
}
async function handleDeleteUser(id: string) {
  await api.delete(`api/CRUDTest/${id}`);
  loadData();
}
async function handleClickOption(btn: btnType, data: user) {
  btn.status === 'edit' ? editUser(data) : deleteUser(data.id || '');
}

onMounted(async () => {
  loadData();
});
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
