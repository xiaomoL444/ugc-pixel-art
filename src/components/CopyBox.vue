<template>
    <button v-on:click="onClick">复制</button>
    {{ content }}
</template>

<script setup>
import { defineProps } from 'vue';
import { toast } from 'vue-sonner';

const props = defineProps({
    content: String  // 声明父组件传递的 prop
})

function onClick() {
    console.log(props.content);

    navigator.clipboard.writeText(props.content)
  .then(() => {
    toast.success('复制成功')
  })
  .catch(err => {
    toast.error('复制失败'+err)
  })
}
</script>