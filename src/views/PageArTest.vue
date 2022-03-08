<template>
  <el-form ref="formRef" :model="myStore" label-width="150px">

    <el-form-item label="Salary 0">
      <el-input-number v-model="form.name"></el-input-number>
    </el-form-item>

    <el-form-item
        v-for="(salary, index) in myStore.salaries"
        :key="salary"
        :label="'Salary ' + (index + 1)"
        :rules="{
        required: true,
        message: 'Salary can not be null',
        trigger: 'blur',
      }"
    >
      <el-input-number :model="salary"></el-input-number>
      <el-button class="mt-2">Delete</el-button>
    </el-form-item>

    <el-form-item>
      <el-button type="info" @click="addMoreSalary">Add Salary</el-button>
    </el-form-item>


    <el-form-item label="Bonus">
      <el-input-number v-model="myStore.test"></el-input-number>
    </el-form-item>

    <el-form-item>
      <el-button type="warning" @click="addMoreSalary">Add Bonus</el-button>
    </el-form-item>


    <el-form-item label="Tax year">
      <el-select v-model="myStore.fiscalYear" placeholder="please select your tax year">
        <el-option v-for="n in 15" :key="currentYear-n" :label="makeLabel(currentYear, n)" :value="makeLabel(currentYear, n)"></el-option>
      </el-select>
    </el-form-item>


    <el-form-item label="Some opt">
      <el-switch v-model="myStore.isOkay"></el-switch>
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
  salaries: [55, 77],
  bonuses: [],
  selectedOpt: [],
  isOkay: true,
  test : 'atiq'
});

const totalSalary = computed(() => {
  return myStore.salaries.length ? myStore.salaries.reduce((a, b)=>a+b, 0) : 0
});

const someOptions = [
    'Option 1',
    'Option 2',
    'Option 3',
    'Option 4'
];

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
