<template>
  <q-page style="min-height: 10px;">
		<div v-if="users == '' ">
				<div class="q-gutter-md" style="max-width: 300px">
						<q-input v-model="username" label="User Name" />
						<q-input v-model="room" label="Room" />
					<q-btn @click="connexion" color="primary" label="Connexion" />
				</div>
		</div>
		<div v-else>
			<div id="message" style="max-height: 500px; min-height: 500px; overflow:auto">
				<ol class='chat' v-for="message in messages" :key="message.id">
					<li :class="message.username == username ? 'self':'other'">
						<div class="avatar"><img :src="'https://eu.ui-avatars.com/api/?name='+message.username" draggable="false"></div>
						<div class="msg"><p>{{message.msg}}</p><time>{{message.date}}</time><br></div>
					</li>
				</ol>
			</div>
			<div class="q-pa-md q-gutter-sm" style="max-width: 500px;">
					<q-input v-model="msg" label="Message" />
					<q-btn round color="secondary" icon="send" @click="sendMsg" />
			</div>
		</div>
		
  </q-page>
</template>

<script>
import setSocket from 'src/mixins/socket'

export default {
  name: 'PageIndex',
  mixins: [setSocket],
  data() {
    return {
			username:"",
			room:"",
			users:'',
			msg:'',
			messages:[]
    }
  },
  mounted() {
    this.setSocket()
    this.socket.on('user', (users) => {
      this.users = users
    })
    this.socket.on('reciveMessage', (messages) => {
      this.messages = messages
    })
    
  },
  methods: {
	  connexion(){
			if(this.username != "" && this.room != ""){
				this.socket.emit('connexion', this.username, this.room)
			}else{
				alert("Please enter Username and Room")
			}
		},
		sendMsg(){
			const date = new Date()
			this.messages.push({id: this.messages.length + 1, username: this.username, msg:this.msg, date:`${date.getDate()}/${date.getMonth()+1}/${date.getUTCFullYear()} ${date.getHours()}:${date.getMinutes()}`})
			this.socket.emit('sendMessage', this.room, this.messages)
			this.msg = ""
		},
    disconnect() {
      this.socket.emit('deco', this.room)
      alert('Lobby disconected')
      location.reload()
    },
  },
}
</script>

<style>

.chat {
    margin-top: 60px;
    padding-left: 0rem;
    margin-bottom: 10px;
    max-width:1200px;
}
.chat li {
    padding: 0.5rem;
    overflow: hidden;
    display: flex;
}
.chat .avatar {
    width: 40px;
    height: 40px;
    position: relative;
    display: block;
    margin : 5px;
    border-radius: 50%;
    background-color: #343434;
    overflow:hidden;
}
.chat .avatar img {
    width: 40px;
    border-radius: 50%;
    background-color: #C4C4C4;
}
.other .msg {
    order: 1;
    border-top-left-radius: 0px;
}
.other:before {
    content: "";
    position: relative;
    top: 0px;
    right: 0px;
    left: 40px;
    width: 0px;
    height: 0px;
    border-left-color: transparent;
    border-bottom-color: transparent;
}

.self {
    justify-content: flex-end;
    align-items: flex-end;
}

.self .avatar {
    order: 2;
}
.msg {
    background: #3E3E3E;
    min-width: 50px;
    max-width: 50%;
    padding: 10px;
    border-radius: 2px;
    box-shadow: 0px 2px 0px rgba(0, 0, 0, 0.07);
    overflow: hidden;
}
.msg p {
    font-size: 0.8rem;
    margin: 0 0 0.2rem 0;
    color: white;
    word-wrap: break-word;

}
.msg img {
    position: relative;
    display: block;
    width: 450px;
    border-radius: 5px;
    transition: all .4s cubic-bezier(0.565, -0.260, 0.255, 1.410);
    cursor: default;
    -webkit-touch-callout: none;
}

.msg time {
    font-size: 0.7rem;
    color: #ccc;
    margin-top: 3px;
    float: right;
    cursor: default;
}

</style>
