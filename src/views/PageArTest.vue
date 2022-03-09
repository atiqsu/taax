<template>
  <el-form ref="formRef" :model="myStore" label-width="150px">

    <el-form-item label="Tax year">
      <el-select v-model="myStore.fiscalYear" placeholder="please select your tax year">
        <el-option v-for="n in 15" :key="currentYear-n" :label="makeLabel(currentYear, n)" :value="makeLabel(currentYear, n)"></el-option>
      </el-select>
    </el-form-item>

    <el-form-item label="Detailed Salary">
      <el-switch v-model="myStore.isOkay"></el-switch>
    </el-form-item>

    <el-form-item
        v-for="(salary, index) in myStore.salaries"
        :key="salary.key"
        :label="'Salary ' + (index + 1)"
        :rules="{
        required: true,
        message: 'Salary can not be null',
        trigger: 'blur',
      }"
    >
      <el-input-number v-model="salary.val"></el-input-number>
      <el-button class="mt-2" @click.prevent="delSalary(salary)">Delete</el-button>
    </el-form-item>

    <el-form-item>
      <el-button type="info" @click="addMoreSalary">Add Salary</el-button>
    </el-form-item>


    <el-form-item
        v-for="(bonus, idx) in myStore.bonuses"
        :key="bonus.key"
        :label="'Bonus ' + (idx + 1)"
    >
      <el-input-number v-model="bonus.val"></el-input-number>
      <el-button class="mt-2" @click.prevent="delBonus(bonus)">Delete</el-button>
    </el-form-item>

    <el-form-item>
      <el-button type="warning" @click="addMoreBonus">Add Bonus</el-button>
    </el-form-item>

    <el-form-item label="Some other options">
      <el-checkbox-group v-model="myStore.selectedOpt" min="0" max="3">
        <el-checkbox v-for="some in someOptions" :key="some" :label="some"></el-checkbox>
      </el-checkbox-group>
    </el-form-item>


    <el-form-item>
      <el-button type="primary" @click="onSubmit">Optimize</el-button>
      <el-button>Cancel</el-button>
    </el-form-item>
  </el-form>

  <div>{{myStore}}</div>
  <div>Total salary : {{totalSalary}}</div>
</template>

<script lang="ts" setup>
import { ref, reactive, computed } from 'vue'

// this is not reactive component and it will not update in vue's life cycle
const currentYear = computed(() => new Date().getFullYear())

let makeLabel: (year:number, decrease:number) => string =  (year:number, decrease:number) => year + ' - ' + (year - decrease);

const myStore = reactive({
  fiscalYear: makeLabel(currentYear.value, 1),
  salaries: [{key: Date.now(), val: 0}],
  bonuses: [{key: Date.now(), val: 0}],
  selectedOpt: [],
  isOkay: true,
  test : 'atiq'
});

const totalSalary = computed(() => {
  return myStore.salaries.length ? myStore.salaries.reduce((a, b)=>a+b.val, 0) : 0
});

const someOptions = [
    'Option 1',
    'Option 2',
    'Option 3',
    'Option 4'
];

const addMoreBonus = () => {
  myStore.bonuses.push({
    key: Date.now(),
    val: 0,
  })
}

const addMoreSalary = () => {
  myStore.salaries.push({
    key: Date.now(),
    val: 0,
  })
}

const delSalary = (item) => {
  const index = myStore.salaries.indexOf(item)
  if (index !== -1) {
    myStore.salaries.splice(index, 1)
  }
}

// do not use same name with ref
const form = reactive({
  name: '',
  region: '',
  date1: '',
  date2: '',
  delivery: false,
  type: [],
  resource: '',
  desc: '',
})

const onSubmit = () => {
  console.log('submit!')
}

</script>

<style scoped>
.el-form {
  border: solid 1px red;
  padding: 10px 0;
}
</style>
