<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn color="primary" class="q-mt-md" @click="add()">新增</q-btn>
      </div>

      <q-dialog v-model="addalert">
        <q-card>
          <q-card-section class="q-pt-none">
            Lorem ipsum dolor sit amet consectetur adipisicing elit. Rerum
            repellendus sit voluptate voluptas eveniet porro. Rerum blanditiis
            perferendis totam, ea at omnis vel numquam exercitationem aut, natus
            minima, porro labore.
          </q-card-section>

          <q-card-actions align="right">
            <q-btn flat label="OK" color="primary" v-close-popup />
          </q-card-actions>
        </q-card>
      </q-dialog>

      <q-dialog v-model="edit" persistent>
        <q-card style="min-width: 350px">
          <q-card-section>
            <div class="text-h6">Your name</div>
          </q-card-section>

          <q-card-section class="q-pt-none">
            <q-input dense v-model="tempData.name" autofocus />
          </q-card-section>
          <q-card-section>
            <div class="text-h6">Your age</div>
          </q-card-section>

          <q-card-section class="q-pt-none">
            <q-input dense v-model="tempData.age" autofocus />
          </q-card-section>

          <q-card-actions align="right" class="text-primary">
            <q-btn flat label="Cancel" v-close-popup @click="edit = false" />
            <q-btn flat label="Edit" @click="apiedit" v-close-popup />
          </q-card-actions>
        </q-card>
      </q-dialog>

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
import { QTableProps } from 'quasar';
import { ref } from 'vue';
import { cloneDeep } from 'lodash';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
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
  id: '',
  name: '',
  age: '',
});
const edit = ref(false);
const editID = ref('');
const handleClickOption = async (btn, data) => {
  if (btn.status == 'edit') {
    edit.value = true;
    editID.value = data.id;
  } else if (btn.status == 'delete') {
    const id = data.id;
    try {
      const del = await axios.delete(
        `https://dahua.metcfire.com.tw/api/CRUDTest/${id}`
      );
      if (del.status == 200) {
        console.log('delete');
      }
    } catch (err) {
      console.log('delerr:', err);
    }
  }
};

const apiedit = async () => {
  edit.value = false;
  try {
    tempData.value.id = editID.value;
    const res = await axios.patch(
      'https://dahua.metcfire.com.tw/api/CRUDTest',
      tempData.value
    );
    if (res.status == 200) {
      getdata();
      tempData.value.age = '';
      tempData.value.id = '';
      tempData.value.name = '';
    }
  } catch (err) {
    console.log('patcherr:', err);
  }
};

const addalert = ref(false);
const add = async () => {
  tempData.value.id = 'idd' + blockData.value.length + 1;
  try {
    const res = await axios.post(
      'https://dahua.metcfire.com.tw/api/CRUDTest',
      tempData.value
    );
    if (res.status == 200) {
      addalert.value = true;
    }
  } catch (err) {
    console.log('posterr:', err);
  }
};

const getdata = async () => {
  console.log('1');
  const data = await axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a');
  try {
    console.log('data', data);
    blockData.value = cloneDeep(data.data);
    console.log('3');
    console.log('b:', blockData.value);
  } catch (err) {
    console.log('getapierr:', err);
  }
};
const init = () => {
  getdata();
};
init();
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
