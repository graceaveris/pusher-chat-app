<template>
<div>
  <div v-if="activeUser.name" class="home">
    <div class="link-bar padding-1">
      <h3>Invite friends to chat by sharing this link <a :href="url" target="blank">{{url}}</a></h3>
    </div>

    <div class="feed padding-1">
      <h1 class="pb-1">Chat Feed</h1>
      <div class="feed-messages" style="overflow-y: scroll; height:400px; position: relative; bottom:0;">
        <div v-for="message in messages" :key="message">
          <Message :message="message" :activeUser="activeUser.name"/>
        </div>
      </div>
      <div class="feed-input">
        <input type="text" ref="message" placeholder="Enter chat message"/>
        <button v-on:click="sendMessage()">Send</button>
      </div>
    </div>

    <div class="users padding-1">
      <h1 class="pb-1">Users</h1>
      <div class="users-me pb-1 mb-1">
        <h2><User :user="activeUser"/></h2>
        <h2 class="ml-2"> - you </h2>
      </div>

      <div class="users-other">
        <div v-for="user in getOtherUsers" :key="user">
          <h2><User :user="user"/></h2>
       </div>
      </div>
    </div>
  </div>
  <div v-else class="home-wall">
   <h1> Add username to enter chatroom</h1>
    <div>
      <input type="text" ref="input" placeholder="enter new username"/>
      <button v-on:click="addName()">submit</button>
    </div>
  </div>
  </div>

</template>

<script>
// Import ChannelDetails component created in diff file
import ChannelDetails from '@/components/ChannelDetails'
//components
import User from '@/components/User'
import Message from '@/components/Message'
export default {
  name: 'home',
  components: { User, Message },
  data () {
    return {

      users: [],
      messages: [],
      activeUser: {
        id: null,
        name: null,
      },
      //channel id
      presenceid: null,
      //url for current channel
      url: null
    }
  },
  created () {
    console.log('created entered')
    this.fetchData()
  },
  computed: {
    getOtherUsers() {
      return this.users.filter((user) => {
        return user.id != this.activeUser.id;
      })
    }
  },
  methods: {
    fetchData () { //WORKING
    // Sets the data instance presenceid variable to the result of the getUniqueId function
    this.presenceid = this.getUniqueId()
    // This checks if there's no presence ID in the URL via the checkPresenceID function and appends the presenceid to the current URL so that we can have the URL end with a parameter
    if (!this.checkPresenceID()) {
      var separator = (window.location.href.indexOf('?') === -1) ? '?' : '&'
      window.location.href = window.location.href + separator + this.presenceid
    }
    // Sets the data instance url variable to the current URL.
    this.url = window.location.href
    // The channel variable is set to to the subscribeToPusher function in ChannelDetails.
    let channel = ChannelDetails.subscribeToPusher()
    console.log(channel)

  // SUBSCRIPTION SUCCEEDED: Triggered once a subscription has been made to a presence channel, exposing the members object to the new user
    channel.bind('pusher:subscription_succeeded', members => {
       console.log('sub succeed', members)
        // add own id
        this.activeUser.id = channel.members.me.id
        // add the existing members
        members.each((member) => {
          //if it is the current user
            this.users.push(this.createMember(member.id))
          })
        })

    //MEMBER ADDED: Triggered when a new user joins. Exposes the new member object to all
    channel.bind('pusher:member_added', newMember => {
      console.log('sub added', newMember)
      this.users.push(this.createMember(newMember.id))
      //in return, user sends their username back
      this.broadCastName()
    })

  // MEMBER REMOVED: When someone leaves the channel.
    channel.bind('pusher:member_removed', member => {
      this.users.forEach((user, index) => {
        if (user.id === member.id) {
          this.users.splice(index, 1)
          this.sendAlert(user.name, 'has left the room')
        }
      })

    })

  // CLIENT SEND NEWUSER NAME: existing users recieve new name// WORKING!!!!
    channel.bind('client-send-name', (res) => {
      this.users.forEach((user) => {
        if (user.id === res.id) {
          user.name = res.name
        }
      })
     })

    channel.bind('client-send-message', (res) => {
      this.messages.push(res.message)
    })
  },

///////////////////// LOGIC FOR MEMBER MANAGEMENT /////////////////////

  createMember(id) {
      const memberObject = {}
          memberObject.id = id
          memberObject.name =  ''
      return memberObject
    },

    addName() {
      //add it to the users object
      this.users.forEach((user) => {
        if (user.id === this.activeUser.id) {
          user.name = this.$refs.input.value
        }
      })
      //add it to personal details
      this.activeUser.name = this.$refs.input.value
      //broadcast new name with corresponding ID
      let channel = ChannelDetails.subscribeToPusher()
      channel.trigger('client-send-name', {id: this.activeUser.id, name: this.activeUser.name })
      //clear fields
      this.$refs.input.value = ""
      //send activity alert to the channel
      this.sendAlert(this.activeUser.name, 'joined the room')
    },

    broadCastName() {
      let channel = ChannelDetails.subscribeToPusher()
      channel.trigger('client-send-name', {id: this.activeUser.id, name: this.activeUser.name })
    },

//// LOGIC FOR MESSAGE MANAGEMENT //////////////////////////////////
    sendMessage() {
      if (this.$refs.message.value) {
        let channel = ChannelDetails.subscribeToPusher()
        let newMessage = {}
        newMessage.message = this.$refs.message.value
        newMessage.author = this.activeUser.name
        newMessage.authorId = this.activeUser.id
        this.messages.push(newMessage)
        channel.trigger('client-send-message', { message: newMessage})

        this.$refs.message.value = ''
      }
    },

    sendAlert(user, alert) {
      let newMessage = {}
      newMessage.message = user + ' has ' + alert
      newMessage.author = "system"
      newMessage.authorId = 0
      let channel = ChannelDetails.subscribeToPusher()
      channel.trigger('client-send-message', { message: newMessage})
    },

//////// LOGIC FOR CHANNEL AND URL /////////////////////////////////
    //generates random alphanumeric characters and adds a prefix of id= to the result.
    getUniqueId () {
      return 'id=' + Math.random().toString(36).substr(2, 8)
    },

    //This function checks the address bar of the browser for URL parameters
    checkPresenceID () {
      let getQueryString = (field, url) => {
        let href = url ? url : window.location.href
        let reg = new RegExp('[?&]' + field + '=([^&#]*)', 'i')
        let string = reg.exec(href)
        return string ? string[1] : null
      }
      let id = getQueryString('id')
      return id
    },

  }
}
</script>

<style scoped>

h1 {
  font-size: 1.8rem;
  margin: 4px;
}

h2 {
  font-size: 1.2rem;
}

button {
  padding: 5px 20px;
  background-color: rgb(12, 156, 137);
  color: white;
  font-weight: 600;
  border: none;
  border-radius: 30px;
  font-size: .8rem;
  margin: 8px;
}

input {
  padding: 5px;
  font-size: .8rem;
  margin: 8px;
}

.pb-1 {
  padding-bottom: 8px;
}

.pb-2 {
  padding-bottom: 4px;
}

.mb-1 {
  margin-bottom: 8px;
}

.mb-2 {
  margin-bottom: 4px;
}

.ml-2 {
  margin-left: 8px;
}

.padding-1 {
  padding: 20px 25px;
}
.padding-2 {
  padding: 8px;
}
.padding-3 {
  padding: 4px;
}
.home-wall {
  text-align: center;
  padding: 30px;
}

.home {
  display: grid;
  grid-template-areas:
    'link-bar link-bar link-bar link-bar link-bar link-bar'
    'feed feed feed feed users users';
  grid-gap: 10px;
  padding: 10px;
}
.link-bar {
  grid-area: link-bar;
  background-color: whitesmoke;
  text-align: center;
}
.link-bar a {
  padding-left: 6px;
  color: rgb(12, 156, 137);
}

.feed {
  grid-area: feed;
  background-color: whitesmoke;
  min-height: 60vh;
}

.feed-input {
  bottom: 0%;
  width: 100%;
  padding: 20px;
}

.feed-input input {
    width: 80%;
}


.feed-messages {
  padding-bottom:
    10px;
    height: 80%;

}
.users {
  grid-area: users;
  background-color: whitesmoke;
  min-height: 60vh;
}
.users-me {
  color:rgb(12, 156, 137);
  display: flex;
}

</style>
