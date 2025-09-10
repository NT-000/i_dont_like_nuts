<script setup>

const {text} = defineProps({
  text: String
})

</script>

<template>
<button class="btn">
  {{text}}
</button>
</template>

<style scoped>

.btn{

}

.isDelete{
  background-color: red;
}
</style>