<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">

        <form @submit.prevent.stop="addNewdata" class="q-gutter-md">
          <!-- @reset.prevent.stop="onReset" -->
          <q-input ref="nameRef" filled v-model="tempData.name" label="姓名 *" lazy-rules :rules="nameRule" />

          <q-input ref="ageRef" filled type="number" v-model="tempData.age" label="年齡 *" lazy-rules :rules="ageRule" />

          <div>
            <q-btn :label="editMode.length > 0 ? '更新' : '新增'" type="submit" color="primary" class="q-mt-md" />
            <!-- <q-btn label="Reset" type="reset" color="primary" flat class="q-ml-sm" /> -->
          </div>
        </form>


      </div>

      <q-table flat bordered ref="tableRef" :rows="blockData" :columns="(tableConfig as QTableProps['columns'])"
        row-key="id" hide-pagination separator="cell" :rows-per-page-options="[0]">
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
            <q-td v-for="col in props.cols" :key="col.name" :props="props" style="min-width: 120px">
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn @click="handleClickOption(btn, props.row)" v-for="(btn, index) in tableButtons" :key="index"
                size="sm" color="grey-6" round dense :icon="btn.icon" class="q-ml-md" padding="5px 5px">

                <q-tooltip transition-show="scale" transition-hide="scale" anchor="top middle" self="bottom middle"
                  :offset="[10, 10]">
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div class="full-width row flex-center items-center text-primary q-gutter-sm" style="font-size: 18px">
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

const $q = useQuasar();




function confirm() {
  $q.dialog({
    dark: true,
    title: 'Alert',
    message: 'Some message'
  }).onOk(() => {
    // console.log('OK')
  }).onCancel(() => {
    // console.log('Cancel')
  }).onDismiss(() => {
    // console.log('I am triggered on both OK and Cancel')
  })
}

const getApi = "https://dahua.metcfire.com.tw/api/CRUDTest/a"; //查詢  get
const addApi = "https://dahua.metcfire.com.tw/api/CRUDTest"; // post 新增
const fixApi = "https://dahua.metcfire.com.tw/api/CRUDTest"; //patch
const deleteApi = "https://dahua.metcfire.com.tw/api/CRUDTest/"; // 後面要 {id} //刪除  delete



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
  name: '',
  age: '',
});

const editMode = ref(0);

async function handleClickOption(btn, data) {
  // ...
  // console.log();
  // console.log(data);
  // console.log(data.id);
  if (btn.icon == "edit") {
    editMode.value = data.id;
    tempData.value.name = "";
    tempData.value.age = "";
  } else {
    // confirm();
    console.log(deleteApi+data.id);
    const data2 = await axiosdata(`${deleteApi}${data.id}`, "delete", "")
    const data3 = await getAlldata();
  }
}








const axiosdata = async (url, method, data) => {
  return await axios({
    method: method,
    url,
    data,
    // headers: {
    //   Authorization: "Bearer " + getToken(),
    // },
  });
}
const nameRef = ref(null)
const ageRef = ref(null)
const addNewdata = async () => {
  const name = tempData.value.name;
  const age = tempData.value.age;
  nameRef.value.validate()
  ageRef.value.validate()

  if (nameRef.value.hasError || ageRef.value.hasError) {
    // form has error
  }
  console.log(editMode.value.length)
  if (editMode.value.length > 0) {
    // 編輯模式


    const data2 = await axiosdata(fixApi, "PATCH", {
      ID: String(editMode.value),
      name: String(name),
      age: age
    })




    editMode.value = 0;
  } else {
    const { data } = await axiosdata(addApi, "post", {
      name: String(name),
      age: age
    })
    console.log(data)
  }


  const data2 = await getAlldata();
}


// 姓名/年齡規則
const nameRule = [
  val => (val && val.length > 0) || 'Please type something'
]
const ageRule = [
  val => (val && val.length > 0 && /^[1-9]\d*$/.test(val)) || '不得為空和限定輸入數字'
]


// 獲取全資料
const getAlldata = async () => {
  const { data } = await axiosdata(getApi, "get", "");
  blockData.value = data;
  // tempData.value.name = "";
  // tempData.value.age = "";
}




onMounted(async () => {
  getAlldata();

})













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
