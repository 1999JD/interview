<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn color="primary" class="q-mt-md" @click="handleAddPeople">
          新增
        </q-btn>
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
import axios from 'axios';
import { QTableProps, useQuasar } from 'quasar';
import { ref, onMounted } from 'vue';
import editDialog from 'src/components/indexpage/editDialog.vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}

interface EditPersonType {
  id?: string;
  name: string;
  age: number;
}

const $q = useQuasar();

const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
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
  name: '',
  age: '',
});

const api = axios.create({
  baseURL: 'https://dahua.metcfire.com.tw/',
});

// 集中管理 api endpoint 、 資料格式
const getPeople = async () => {
  return api.get('/api/CRUDTest/a');
};

const addPeople = async (data: EditPersonType) => {
  return api.post('/api/CRUDTest', data);
};

const updatePeople = async (data: EditPersonType) => {
  return api.patch('/api/CRUDTest', data);
};

const deletePeople = async (id: string) => {
  return api.delete(`/api/CRUDTest/${id}`);
};

// 處理頁面對 api 的操作及 ui 邏輯
const handleGetPeople = async () => {
  try {
    const response = await getPeople();
    blockData.value = response.data;
  } catch (err) {
    alert(err);
  }
};

const handleAddPeople = async () => {
  try {
    const data = {
      name: tempData.value.name,
      age: Number(tempData.value.age),
    };

    await addPeople(data);
    tempData.value.name = '';
    tempData.value.age = '';
    handleGetPeople();
  } catch (err) {
    alert(err);
  }
};

const handleUpdatePeople = async (data: EditPersonType) => {
  try {
    await updatePeople(data);
    handleGetPeople();
  } catch (err) {
    alert(err);
  }
};

const handleDeletePeople = async (id: string) => {
  try {
    await deletePeople(id);
    handleGetPeople();
  } catch (err) {
    alert(err);
  }
};

function handleClickOption(
  btn: btnType,
  data: {
    id: string;
    name: string;
    age: number;
  }
) {
  if (btn.status === 'edit') {
    $q.dialog({
      component: editDialog,

      // props forwarded to your custom component
      componentProps: {
        name: data.name,
        age: Number(data.age),
        id: data.id,
        api: api,
        // ...more..props...
      },
    }).onOk((data) => {
      console.log(data);
      handleUpdatePeople(data);
      console.log('OK');
    });
  } else if (btn.status === 'delete') {
    $q.dialog({
      title: 'Confirm',
      message: '您確定要刪除此項目?',
      cancel: true,
      persistent: true,
    }).onOk(() => {
      handleDeletePeople(data.id);
    });
  }
}

onMounted(() => {
  handleGetPeople();
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
