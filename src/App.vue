    <template>
      <div id="app">
        <router-view></router-view>
      </div>
    </template>

    <script>
    export default {
      name: 'app'
    }
    </script>

    <style>
    a
      html {
      }
      body {
        padding: 20px;
      }
      #app {
        height: 100vh;
        font-family: montserrat;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        color: rgb(20, 20, 20);
      }
      .fade-enter-active, .fade-leave-active {
        transition: opacity .5s
      }
      .fade-enter, .fade-leave-to /* .fade-leave-active in <2.1.8 */ {
        opacity: 0
      }
    </style>
