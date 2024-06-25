<template>
  <div>
    <form @submit.prevent="validateInput">
      <div v-for="(item, index) in myArray" :key="index">
        <input v-model="myArray[index]" type="text" />
      </div>
      <button type="button" @click="addItem">添加项</button>
      <button type="submit">提交</button>
    </form>
    <div v-if="errorMessage">
      <p>{{ errorMessage }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import * as Yup from "yup";

const myArray = ref([""]);
const errorMessage = ref("");

const schema = Yup.object().shape({
  myArray: Yup.array()
    .of(
      Yup.string()
        .required("每个元素都必须是一个字符串")
        .min(3, "字符串长度至少为3")
    )
    .required("数组是必须的")
    .min(1, "数组中至少应有一个元素"),
});

const validateInput = async () => {
  try {
    await schema.validate({ myArray: myArray.value });
    errorMessage.value = "";
    // 在这里处理有效的输入
    console.log("输入有效:", myArray.value);
  } catch (err) {
    errorMessage.value = err.message;
    console.error("输入无效:", err);
  }
};

const addItem = () => {
  myArray.value.push("");
};
</script>
