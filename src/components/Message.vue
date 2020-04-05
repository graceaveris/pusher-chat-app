<template>
<transition name="fade">

  <div :class="getClass">
   <p><span>{{getName}}</span>{{message.message}}</p>
  </div>
</transition>
</template>

<style scoped>
 p {
   margin: 3px;
  padding: 3px 15px;
  background-color: white;
  font-size: 16px;
  display: inline-block;
  border-radius: 4px;
}

.active-user {
  font-size: 16px;
  text-align: end;
 color:  rgb(12, 156, 137);
}

.system p {
  background-color: whitesmoke !important;
  font-size: 12px;
  font-style: italic;
  padding-left: 3px !important;
}
</style>

<script>
export default {
  name: 'Message',
  props: {
    message: {
     type: Object,
     default: null,
    },
    activeUser: {
     type: String,
     default: null,
    }
  },
  computed: {
    getClass() {
    if (this.message.author === 'system') {
        console.log(this.message.author)
        return 'system'
      } else if (this.message.author === this.activeUser) {
        console.log('message working', this.message.author)
        return 'active-user'
      }
    },
    getName() {
      if (this.message.author !== this.activeUser) {
        return this.message.author + " - "
      }
    }
  },
}
</script>
