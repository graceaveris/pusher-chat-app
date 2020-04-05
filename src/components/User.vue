<template>
<transition name="fade">
  <div class="user">
   <p>{{user.name}}</p>
  </div>
</transition>
</template>


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
