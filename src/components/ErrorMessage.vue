<template>
<div>
  <p>{{message}}</p>
</div>
</template>

<script>
export default {
  name: "ErrorMessage",
  props: ['message']
}
</script>

<style scoped>
div {
  background-color: #e31c3d;
  color: #ffffff;
  font-weight: bold;
}
</style>

