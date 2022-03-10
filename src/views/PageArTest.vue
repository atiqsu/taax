<template>
  <el-form ref="formRef" :model="myStore" label-width="150px">

    <el-form-item label="Tax year">
      <el-select v-model="myStore.fiscalYear" placeholder="please select your tax year">
        <el-option v-for="n in 15" :key="currentYear-n" :label="makeLabel(currentYear, n)" :value="makeLabel(currentYear, n)"></el-option>
      </el-select>
    </el-form-item>

    <el-form-item label="Detailed Salary">
      <el-switch v-model="myStore.isOkay" inactive-text="Salary details" active-text="Salary/month"></el-switch>
    </el-form-item>

    <el-form-item
        v-for="(salary, index) in myStore.salaries"
        :key="salary.key"
        :label="'Salary ' + (myStore.isOkay ? '' : (index + 1))"
        :rules="{
        required: true,
        message: 'Salary can not be null',
        trigger: 'blur',
      }"
    >
      <el-input-number v-model="salary.val"></el-input-number>
      <el-button class="mt-2" @click.prevent="delSalary(salary)" v-if="enableSalaryDelBtn">Delete</el-button>
    </el-form-item>

    <el-form-item>
      <el-button type="info" @click="addMoreSalary" v-if="!myStore.isOkay">Add Salary</el-button>
    </el-form-item>


    <el-form-item
        v-for="(bonus, idx) in myStore.bonuses"
        :key="bonus.key"
        :label="'Bonus ' + (idx + 1)"
    >
      <el-input-number v-model="bonus.val"></el-input-number>
      <el-button class="mt-2" @click.prevent="delBonus(bonus)" v-if="enableBonusDelBtn">Delete</el-button>
    </el-form-item>

    <el-form-item>
      <el-button type="warning" @click="addMoreBonus">Add Bonus</el-button>
    </el-form-item>

    <el-form-item label="Make editable">
      <el-checkbox-group v-model="myStore.selectedOpt" min="0" max="3">
        <el-checkbox v-for="some in someOptions" :key="some" :label="some"></el-checkbox>
      </el-checkbox-group>
    </el-form-item>

    <el-form-item label="Basic percentage">
      <el-input-number v-model="myStore.pctRate" precision="2" step=".01"></el-input-number>
    </el-form-item>

    <el-form-item label="Conveyance">
      <el-input-number v-model="myStore.conveyance" :disabled="!myStore.selectedOpt.includes(editConv)"></el-input-number>
    </el-form-item>

    <el-form-item label="Medical">
      <el-input-number v-model="myStore.medicalAllowance" :disabled="!myStore.selectedOpt.includes(editMed)"></el-input-number>
    </el-form-item>

    <el-form-item>
      <el-button type="primary" @click="onSubmit">Optimize</el-button>
      <el-button>Cancel</el-button>
    </el-form-item>
  </el-form>

  <br>
  <br>


  <el-table :data="tblItems" border stripe show-summary sum-text="Total">
    <el-table-column prop="area" fixed label="Area" width="200px" />
    <el-table-column prop="amount" label="Amount" />
    <el-table-column prop="exemption" label="Max. Exemption" />
    <el-table-column prop="taxable" label="Taxable" width="300px" />
  </el-table>


  <br>
  <br>

  <el-table :data="taxSlabs" border stripe>
    <el-table-column prop="lbl" label="Slab" width="200px"></el-table-column>
    <el-table-column prop="taxPctTxt" label="Tax%"></el-table-column>
    <el-table-column prop="amount" label="Amount"></el-table-column>
    <el-table-column prop="tax" label="Tax"></el-table-column>
  </el-table>



  <br>
  <br>
  <br>

  <div>Total tax need to pay: {{payableTotalTax}}</div>
  <div>Total salary : {{totalSalary}}</div>
  <div>Total bonus : {{totalBonus}}</div>
  <div>Total taxable : {{totalTaxableAmount}}</div>
</template>

<script lang="ts" setup>
import { ref, reactive, computed } from 'vue'
import getTimerCount = jest.getTimerCount;

// this is not reactive component and it will not update in vue's life cycle
const currentYear = computed(() => new Date().getFullYear())

let makeLabel: (year:number, decrease:number) => string =  (year:number, decrease:number) => year + ' - ' + (year - decrease);

const myStore = reactive({
  fiscalYear: makeLabel(currentYear.value, 1),
  salaries: [{key: Date.now(), val: 0}],
  bonuses: [{key: Date.now(), val: 0}],
  selectedOpt: [],
  isOkay: true,
  conveyance: 30000,
  medicalAllowance: 5000,
  taxableMed: 0,
  taxableConv:0,
  pctRate: 60,
  test : 'atiq'
});


const totalSalary = computed(() => {
  if(myStore.isOkay) {
    return myStore.salaries[0].val * 12;
  }
  return myStore.salaries.length ? myStore.salaries.reduce((a, b)=>a+b.val, 0) : 0
});

const totalBonus = computed(() => {
  return myStore.bonuses.length ? myStore.bonuses.reduce((a, b)=>a+b.val, 0) : 0
});

const enableBonusDelBtn = computed(()=>{ return myStore.bonuses.length > 1});

const enableSalaryDelBtn = computed(()=> myStore.salaries.length > 1);

const basicAmount = computed(()=> Math.ceil(totalSalary.value * myStore.pctRate /100) );
const convAmount = computed(()=> myStore.conveyance) ;
const rentAmount = computed(()=> totalSalary.value - basicAmount.value - convAmount.value - calcMedMaxExemption.value) ;

const calcRentMaxExemption = computed(()=> {
  let c = Math.ceil(basicAmount.value * .5);

  if(c > 300000) {

    return 300000;
  }

  return c;
}) ;
const calcMedMaxExemption = computed(()=> {
  let c = Math.ceil(basicAmount.value * .1);

  if(c > 120000) {
    return 120000
  }

  return c;
}) ;

const calcTaxableRent = computed(()=> {
  return (rentAmount.value <= calcRentMaxExemption.value ? 0 : (rentAmount.value - calcRentMaxExemption.value))
}) ;

const totalTaxableAmount = computed(()=>basicAmount.value + calcTaxableRent.value + totalBonus.value + myStore.taxableMed + myStore.taxableConv);

const editConv = 'Edit Conveyance';
const editMed = 'Edit Medical allowance';

const someOptions = [
    editConv,
    editMed,
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

  if(myStore.salaries.length === 1) return ;

  const index = myStore.salaries.indexOf(item)
  if (index !== -1) {
    myStore.salaries.splice(index, 1)
  }
}

const delBonus = (item) => {
  const index = myStore.bonuses.indexOf(item)
  if (index !== -1) {
    myStore.bonuses.splice(index, 1)
  }
}


const calcSlab1Amount = computed(()=> totalTaxableAmount.value > 300000 ? 300000: totalTaxableAmount.value);
const calcSlab2Amount = computed(()=> {
  if(totalTaxableAmount.value > calcSlab1Amount.value) {
    if(totalTaxableAmount.value - calcSlab1Amount.value - 100000 > 0 ) {
      return 100000;
    }

    return totalTaxableAmount.value - calcSlab1Amount.value;
  }

  return 0;
});

const calcSlab3Amount = computed(()=> {
  let c = calcSlab1Amount.value + calcSlab2Amount.value;

  if(totalTaxableAmount.value - c > 0) {
    c = totalTaxableAmount.value - c;

    if(c > 300000) {
      return 300000;
    }

    return c;
  }

  return 0;
});

const calcSlab4Amount = computed(()=> {
  let c = calcSlab1Amount.value + calcSlab2Amount.value + calcSlab3Amount.value;

  if(totalTaxableAmount.value - c > 0) {
    c = totalTaxableAmount.value - c;

    if(c > 400000) {
      return 400000;
    }

    return c;
  }

  return 0;
});

const calcSlab5Amount = computed(()=> {
  let c = calcSlab1Amount.value + calcSlab2Amount.value + calcSlab3Amount.value + calcSlab4Amount.value;

  if(totalTaxableAmount.value - c > 0) {
    c = totalTaxableAmount.value - c;

    if(c > 500000) {
      return 500000;
    }

    return c;
  }

  return 0;
});


const calcSlab6Amount = computed(()=> {
  let c = calcSlab1Amount.value + calcSlab2Amount.value + calcSlab3Amount.value + calcSlab4Amount.value + calcSlab5Amount.value;

  if(totalTaxableAmount.value - c > 0) {
    return totalTaxableAmount.value - c;
  }

  return 0;
});

const calcSlab2Tax = computed(()=> calcSlab2Amount.value > 0 ? calcSlab2Amount.value * .05 : 0)
const calcSlab3Tax = computed(()=> calcSlab3Amount.value > 0 ? calcSlab3Amount.value * .10 : 0)
const calcSlab4Tax = computed(()=> calcSlab4Amount.value > 0 ? calcSlab4Amount.value * .15 : 0)
const calcSlab5Tax = computed(()=> calcSlab5Amount.value > 0 ? calcSlab5Amount.value * .20 : 0)
const calcSlab6Tax = computed(()=> calcSlab6Amount.value > 0 ? calcSlab6Amount.value * .25 : 0)
const payableTotalTax = computed(()=> calcSlab2Tax.value + calcSlab3Tax.value + calcSlab4Tax.value + calcSlab5Tax.value + calcSlab5Tax.value)


const tblItems = [
  {
    area: 'Basic salary',
    amount: basicAmount,
    exemption: 0,
    taxable: basicAmount
  },
  {
    area: 'House rent',
    amount: rentAmount,
    exemption: calcRentMaxExemption,
    taxable: calcTaxableRent
  },
  {
    area: 'Medical allowance',
    amount: calcMedMaxExemption,
    exemption: calcMedMaxExemption,
    taxable: myStore.taxableMed
  },
  {
    area: 'Conveyance',
    amount: convAmount,
    exemption: 30000,
    taxable: myStore.taxableConv
  },
  {
    area: 'Bonuses',
    amount: totalBonus,
    exemption: 0,
    taxable: totalBonus
  }
];

const taxSlabs = [
  {
    lbl: 'Upto 3,00,000',
    taxPct: 0,
    taxPctTxt: '0%',
    amount: calcSlab1Amount,
    tax: 0,
  },
  {
    lbl: 'On the next 1,00,000',
    taxPct: 5,
    taxPctTxt: '5%',
    amount: calcSlab2Amount,
    tax: calcSlab2Tax,
  },
  {
    lbl: 'Upto the next 3,00,000',
    taxPct: 10,
    taxPctTxt: '10%',
    amount: calcSlab3Amount,
    tax: calcSlab3Tax,
  },
  {
    lbl: 'Upto the next 4,00,000',
    taxPct: 15,
    taxPctTxt: '15%',
    amount: calcSlab4Amount,
    tax: calcSlab4Tax,
  },
  {
    lbl: 'Upto the next 5,00,000',
    taxPct: 20,
    taxPctTxt: '20%',
    amount: calcSlab5Amount,
    tax: calcSlab5Tax,
  },
  {
    lbl: 'Above',
    taxPct: 25,
    taxPctTxt: '25%',
    amount: calcSlab6Amount,
    tax: calcSlab6Tax
  }
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
