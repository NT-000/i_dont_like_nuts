<script setup>

const {isOpen} = defineProps({
  isOpen: Boolean,

})

</script>

<template>

  <teleport to="body">
    <div class="box" v-if="isOpen">
      <div class="dialog">
        <span>Your Badges<i class="fa-solid fa-trophy" style="color: #FFD43B;"></i></span>
        <p>
          <slot/>
        </p>
      </div>
    </div>
  </teleport>

</template>

<style scoped>
.box {
  display: flex;
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.65);
  align-items: center;
  justify-content: center;
}

.dialog {
  background: grey;
  padding: 1rem;
  border-radius: 10px;
  min-width: 350px;
  text-align: center;
  margin-bottom: 20px;
  font-style: italic;
  font-weight: bolder;
}

p {
  margin-bottom: 20px;
}

span {
  display: block;
  justify-content: center;
  align-items: center;
  text-align: center;
}

</style>