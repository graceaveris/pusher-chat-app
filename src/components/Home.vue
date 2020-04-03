<template>
  <div class="home">

    <div class="link-bar">
      <p>Invite friends to chat by sharing this link <a :href="url" target="blank">{{url}}</a></p>
    </div>

    <div class="feed">
      <h3>Chat Feed</h3>
    </div>

    <div class="users">

      <div class="users-me">
        <User :user="activeUser" />
        <form >
          <input type="text" placeholder="enter new username" @onchange="updateName()"/>
          <button>submit</button>
        </form>
      </div>

      <div class="users-other">
        <div v-for="user in getOtherUsers" :key="user">
          <User :user="user"/>
       </div>
      </div>

    </div>

    <div class="footer">
      <h3>Footer</h3>
    </div>
                <!-- <div>
                  <button @click="incrementCount()">add</button>
                </div>
                <div>
                  <button @click="setPlayerName()">Add my name</button>
                </div> -->
  </div>
</template>

<style scoped>

.home {
  display: grid;
  grid-template-areas:
    'link-bar link-bar link-bar link-bar link-bar link-bar'
    'feed feed feed feed users users'
    'footer footer footer footer footer footer';
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
  color: blue;
}

.feed {
  grid-area: feed;
  background-color: whitesmoke;
  min-height: 60vh;
}
.users {
  grid-area: users;
  background-color: whitesmoke;
  min-height: 60vh;
}
.users-me {
  background-color: aquamarine;
}
.users-other {
}

.footer {
  grid-area: footer;
  background-color: whitesmoke;
}
</style>

<script>
// Import ChannelDetails component created in diff file
import ChannelDetails from '@/components/ChannelDetails'
//components
import User from '@/components/User'
export default {
  name: 'home',
  components: { User },
  data () {
    return {
      //PRIVATE DATA (which we recieve from members object or locally)
      //my game
      presenceid: null,
      //PUBLIC DATA (data that is broadcast and shared)
      users: [],
      // This holds the players own details
      activeUser: {
        id: null,
        name: null,
      },
      //url for current channel
      url: null
    }
  },
  created () {
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
    fetchData () {
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

    //BINDING USER EVENTS TO PUSHER
    //MEMBER ADDED: Triggered when a new user joins. Exposes the new member object to all other subscribers to save and use.
    channel.bind('pusher:member_added', newMember => {
      this.players += 1 //add an extra player to player count
      console.log('New Member Joined')
      this.users.push(this.createMember(newMember.id))
    })

  // SUBSCRIPTION SUCCEEDED: Triggered once a subscription has been made to a presence channel, exposing the members object to the new user
    channel.bind('pusher:subscription_succeeded', members => {
        console.log('You have joined the channel!')
        // add the existing members
        members.each((member) => {
          //if it is the current user
          if (member.id === channel.members.me.id) {
            this.users.push(this.createMember(member.id))
            // adds personal details to non-public data
            this.activeUser.id = member.id
            this.activeUser.name = 'Anonymous-' + member.id.substr(2, 5)
          } else {
            //all other users
            this.users.push(this.createMember(member.id))
          }
        })
    })

    // MEMBER REMOVED: When someone leaves the channel.
    channel.bind('pusher:member_removed', member => {
      this.players -= 1
      if (member.count === 1) {
        this.secondplayer = false
      }
    })

      // CLIENT SEND COUNT: updates the count
    channel.bind('client-send-username', (res) => { //{data: this.activeUser})
        this.users.forEach((user, index) => {
          if (user.id === res.id) {
              this.users[index].name = res.name
          }
      })
    })

    // CLIENT SEND COUNT: updates the count
    channel.bind('client-send-count', (res) => {
        this.count = res.data
    })

    // CLIENT SEND NAME: updates the name
    channel.bind('client-send-name', (res) => {
      console.log('entered client-send-name')
      if (this.userid === 1) {
        this.users.user2.name = res.data.user2.name
      } else if (this.userid === 2) {
        this.users.user1.name = res.data.user1.name
      }
    })
    },

///////////////////// END OF FETCH /////////////////////

    //This function simply generates random alphanumeric characters and adds a prefix of id= to the result.
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

    createMember(id) {
    const memberObject = {}
          memberObject.id = id
          memberObject.name = 'Anonymous-' + id.substr(2, 5)
          return memberObject
    },

    incrementCount () {
      let channel = ChannelDetails.subscribeToPusher()
      //incrementing on click
      this.count++
      channel.trigger('client-send-count', {data: this.count})
    },

    updateName () {
      let channel = ChannelDetails.subscribeToPusher()
      //adding a name to user
        if (this.userid === 1) {
          this.users.user1.name = 'Greg'
        } else if (this.userid === 2) {
          this.users.user2.name = 'Jennifer'
        }
      channel.trigger('client-send-name', {data: this.users})
    },
  }
}
</script>
