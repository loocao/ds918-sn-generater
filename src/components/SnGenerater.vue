<template>
  <h1>DS918算号器</h1>

  <Form ref="refForm" :model="formState" :rules="rules" :label-col="{ span: 8 }">
    <FormItem label="SN" name="sn">
      <Input v-model:value="formState.sn" placeholder="输入正确的SN" />
    </FormItem>
    <FormItem label="MAC" name="mac">
      <Input v-model:value="formState.mac" placeholder="输入正确的MAC" />
    </FormItem>
    <FormItem>
      <Button @click="generate">生成</Button>
    </FormItem>
    <FormItem label="新的SN">
      <Input v-model:value="formState.newSn" placeholder="新的SN" readonly />
    </FormItem>
    <FormItem label="新的MAC">
      <Input v-model:value="formState.newMac" placeholder="新的MAC" readonly />
    </FormItem>
  </Form>
</template>

<script setup>
import { Button, Form, FormItem, Input } from 'ant-design-vue'
import { ref, reactive } from 'vue'

const refForm = ref()
const formState = reactive({
  sn: '',
  mac: '',
  newSn: '',
  newMac: '',
})
const rules = {
  sn: [
    { required: true, message: '请输入SN', trigger: 'blur' },
    { len: 13, message: 'SN长度不正确', trigger: 'blur' },
  ],
  mac: [
    { required: true, message: '请输入MAC', trigger: 'blur' },
    { len: 12, message: 'MAC长度不正确', trigger: 'blur' },
  ],
}

function generate() {
  refForm.value
    .validate()
    .then(genSn)
    .catch((error) => {
      console.log('表单验证失败', error)
    })
}

function genSn() {
  const sn = formState.sn
  const mac = formState.mac
  console.log('生成新的SN, SN=' + sn, 'MAC=' + mac)
  // CC
  const cc = sn.slice(0, 2)
  console.log('CC=' + cc)
  // Y
  const y = sn.slice(2, 3)
  console.log('Y=' + y)
  // AAA
  const aaa = sn.slice(7, 10)
  console.log('AAA=' + aaa)
  // BB
  const bb = sn.slice(-2)
  console.log('BB=' + bb)

  // 随机一个数字，赋值给AAA
  const AAA = Math.floor(Math.random() * 1000)

  // 随机BB，BB数值不要太大一般控制在15以内
  const BB = Math.floor(Math.random() * 16)

  // (((1000 * BB + AAA) % CC) + Y) % 10 = #
  const num = (((1000 * BB + AAA) % cc) + y) % 10
  console.log('#=' + num)

  // 新的SN=CCY0PDNAAA#BB
  const formatBB = BB < 10 ? ('0' + BB) : BB
  formState.newSn = cc + y + '0PDN' + AAA + num + formatBB
  console.log('newSn=' + formState.newSn)

  // 计算MAC
  const newMac = calcMac(mac, AAA, BB)
  formState.newMac = ('001132' + newMac).toUpperCase()
  console.log('newMac=' + formState.newMac)
}

function calcMac(mac, AAA, BB) {
  // 获取MAC地址后六位字符
  const macHexLastSix = mac.slice(-6)
  // 将macHexLastSix转换为10进制
  const macDecLastSix = parseInt(macHexLastSix, 16)

  // 计算差值, macDecLastSix - AAA * 2
  const diff = macDecLastSix - AAA * 2
  console.log('diff=' + diff)

  // (1000 * BB + AAA) * 2 + 3869533
  const newMacLastSix = (1000 * BB + AAA) * 2 + diff
  // 将mac转换为16进制
  const macHex = newMacLastSix.toString(16)
  return macHex
}
</script>
