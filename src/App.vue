<template :class="theme">
<NavBar :room="currentRoom" class="navbar" @roomSearch="roomChange" />
<MessageRender class="messages" @sendMessage="sendMessage" :messageList="messageList" />
<UsersOnline class="users" />
<transition-group name="fade">
  <LoginModal class="modal" @logIn="logIn" v-if="!isLoggedIn" />
  <div class="modal-blocker" v-if="!isLoggedIn"></div>
</transition-group>
</template>

<script>
// These global variables are used to provide API and redirect routes.
window.serverHost = "https://modchatserver.herokuapp.com"; // please reset to https://modchatserver.herokuapp.com after testing or development
window.clientHost = "https://modchat.micahlindley.com"; // please reset to https://modchat.micahlindley.com after testing or development
import {
  io
} from "socket.io-client";
import MessageRender from './components/MessageRender.vue';
import NavBar from './components/NavBar.vue';
import UsersOnline from './components/UsersOnline.vue';
import LoginModal from './components/LoginModal.vue';
console.log('Server host:', window.serverHost);
const socket = io(window.serverHost);
export default {
  name: 'App',
  components: {
    NavBar,
    MessageRender,
    UsersOnline,
    LoginModal
  },
  methods: {
    roomChange(name) {
      socket.disconnect();
      this.currentRoom = name.substring(1);
      this.messageList = [];
      socket.connect();
      console.log('changeRoom:' + name)
    },
    enableModal() {
      this.modalEnabled = true;
    },
    sendMessage(msg) {
      socket.emit("chat", {
        type: 'text',
        content: msg,
        token: this.user.token
      });
    }
  },
  data() {

    /*
     */
    if (!window.localStorage.getItem('user')) {
      window.localStorage.setItem('user', `{
        "name": "Unauthed User",
        "token": 0,
        "password": ""
      }`);
      window.location.reload();
    }
    return {
      currentRoom: 'general',
      user: JSON.parse(window.localStorage.getItem('user')),
      messageList: []
    }
  },
  mounted() {
    let that = this;
    socket.on("connect", () => {
      console.log('Connected to server');
      socket.emit('authentication', {
        username: that.user.name,
        password: that.user.password
      });
      console.log('Submitted auth');
      socket.on('authenticated', function() {
        console.log('Successfully authed');
        socket.emit("joinRoom", {
          "username": that.user.name,
          "roomname": that.currentRoom
        });
        socket.on("message", (obj) => {
          console.log('Recieved a message');
          that.messageList.unshift(obj);
          console.log(that.messageList);
        })
      });
    });

  },
  computed: {
    isLoggedIn() {
      if (window.localStorage.getItem('user')) {
        if (JSON.parse(window.localStorage.getItem('user')).name == 'Unauthed User') {
          return false;
        } else {
          return true;
        }
      } else {
        return false;
      }
    }
  }
}
</script>

<style>
#app {
  font-family: 'Inter', Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  background: var(--bg-primary);
  height: 100vh;
  display: grid;
  grid-template-columns: 1fr 240px;
  grid-template-rows: 70px 1fr 100px;
  overflow: hidden;
}

.messages {
  grid-row: 2 / 3;
}

.navbar {
  grid-row: 1 / 2;
  grid-column: 1 / 3;
  box-shadow: var(--shadow);
}

.users {
  grid-row: 1 / 4;
  grid-column: 2 / 3;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.modal-blocker {
  height: 100vh;
  width: 100%;
  position: absolute;
  top: 0;
  left: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 3;
  backdrop-filter: blur(4px);
}

.modal {
  z-index: 4;
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

@media only screen and (max-width: 700px) {
  .messages {
    grid-column: 1 / 3;
  }
}
</style>
