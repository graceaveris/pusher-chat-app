<template>
  <div class="user">
   <p>{{user.name}}</p>
  </div>
</template>

<style scoped>
.user {
  font-size: 16px;
}
</style>

<script>
export default {
  name: 'User',
  props: {
    user: {
     type: Object,
     default: null,
    }
  },
}
</script>
