<template>
  <q-page style="min-height: 10px;">
		<div v-if="users == '' ">
				<div class="q-gutter-md connexion" style="max-width: 300px; display:block !important">
						<q-input color="primary" bg-color="white" outlined v-model="username" label="User Name" />
						<q-input color="primary" bg-color="white" outlined  v-model="room" label="Room" />
						<center>
							<q-btn style="" @click="connexion" color="primary" label="Connexion" />
						</center>
				</div>
		</div>
		<div v-else>
			<div id="message" style="max-height: 600px; min-height: 600px; overflow:auto; background-color:#545454">
				<ol class='chat' v-for="message in messages" :key="message.id">
					<li :class="message.username == username ? 'self':'other'">
						<div class="avatar"><img :src="'https://eu.ui-avatars.com/api/?name='+message.username" draggable="false"></div>
						<div class="msg"><p>{{message.msg}}</p><time>{{message.date}}</time><br></div>
					</li>
				</ol>
			</div>
			<div class="q-pa-md q-gutter-sm" style="">
				<q-input v-model="msg" bottom-slots label="Message">
        <template v-slot:before>
          <q-avatar>
            <img :src="'https://eu.ui-avatars.com/api/?name='+username">
          </q-avatar>
        </template>

        <template v-slot:after>
          <q-btn round dense flat icon="send" @click="sendMsg" />
        </template>
      	</q-input>
			</div>
			<div class="q-pa-md q-gutter-sm">
				<q-btn v-if="!isCall" @click="onJoin" push color="primary" label="Join call" icon-right="call"/>
				<q-btn v-if="isCall" @click="onLeave" color="red" label="Leave" icon-right="call_end"/>
				<q-btn v-if="isCall" @click="onShareScreen" push color="green" label="Share Screen" icon-right="desktop_windows"/>
			</div>
		</div>
		<vue-webrtc width="100%" :roomId="room" ref="webrtc">
		</vue-webrtc>
		<img :src="img" class="img-responsive" />

		
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
			messages:[],
			img:'',
			isCall: true,
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
				this.onJoin()
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
		onJoin() {
        this.$refs.webrtc.join();
				this.isCall = true
      },
      onLeave() {
				this.$refs.webrtc.leave();
				this.isCall = false
      },
      onShareScreen() {
        this.img = this.$refs.webrtc.shareScreen();
      },
  },
}
</script>

<style>

.connexion{
	margin: auto;
	background-color: #343434;
	padding: 25px;;
}

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
