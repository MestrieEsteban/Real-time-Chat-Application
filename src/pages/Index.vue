<template >
  <q-page>
		<div v-if="users == '' ">
			<div class="grid">
				<div class="form login">
					<div class="form__field">
						<label for="login__username"><svg class="icon">
								<use xlink:href="#icon-user"></use>
							</svg><span class="hidden">Username</span></label>
						<input  v-model="username" autocomplete="username" id="login__username" type="text" name="username" class="form__input" placeholder="Username" required>
					</div>
					<div class="form__field">
						<label for="login__password"><svg class="icon">
								<use xlink:href="#icon-lock"></use>
							</svg><span class="hidden">Room</span></label>
						<input v-model="room" id="login__password" type="text" name="password" class="form__input" placeholder="Room" required>
					</div>
					<div class="form__field">
						<input type="submit" @click="connexion" value="Connexion">
					</div>
				</div>
			</div>
			<svg xmlns="http://www.w3.org/2000/svg" class="icons">
				<symbol id="icon-lock" viewBox="0 0 1792 1792">
					<path d="M640 768h512V576q0-106-75-181t-181-75-181 75-75 181v192zm832 96v576q0 40-28 68t-68 28H416q-40 0-68-28t-28-68V864q0-40 28-68t68-28h32V576q0-184 132-316t316-132 316 132 132 316v192h32q40 0 68 28t28 68z" />
				</symbol>
				<symbol id="icon-user" viewBox="0 0 1792 1792">
					<path d="M1600 1405q0 120-73 189.5t-194 69.5H459q-121 0-194-69.5T192 1405q0-53 3.5-103.5t14-109T236 1084t43-97.5 62-81 85.5-53.5T538 832q9 0 42 21.5t74.5 48 108 48T896 971t133.5-21.5 108-48 74.5-48 42-21.5q61 0 111.5 20t85.5 53.5 62 81 43 97.5 26.5 108.5 14 109 3.5 103.5zm-320-893q0 159-112.5 271.5T896 896 624.5 783.5 512 512t112.5-271.5T896 128t271.5 112.5T1280 512z" />
				</symbol>
			</svg>
		</div>
		<div v-else>
			<div class="row">
				<div class="col-12 col-md-8">
					<div class="q-pa-md q-gutter-sm">
						<q-btn v-if="!isCall" @click="onJoin" push color="primary" label="Join call" icon-right="call"/>
						<q-btn v-if="isCall" @click="onLeave" color="red" label="Leave call" icon-right="call_end"/>
						<q-btn v-if="isCall" @click="onShareScreen" push color="green" label="Share Screen" icon-right="desktop_windows"/>
					</div>
					<div class="q-pa-md q-gutter-sm">
						<vue-webrtc cameraHeight=250 style="background-color:#2C3338" width="100%" :roomId="room" ref="webrtc"></vue-webrtc>
						<img v-if="img != ''" :src="img" class="img-responsive" />
					</div>
				</div>
				<div class="col-12 col-md-4">
					<div id="message" style="height: 600px; min-height: 600px; overflow:auto; background-color:#545454">
						<ol class='chat' v-for="message in messages" :key="message.id">
							<li :class="message.username == username ? 'self':'other'">
								<div class="avatar"><img :src="'https://eu.ui-avatars.com/api/?name='+message.username" draggable="false"></div>
								<div class="msg"><p>{{message.msg}}</p><time>{{message.date}}</time><br></div>
							</li>
						</ol>
					</div>
					<div class="q-pa-md q-gutter-sm" style="">
						<q-input rounded outlined :rules="[ val => val.length <= 50 || 'Please use maximum 50 characters']"  label-color="white" color="white" dark ref="mess"  v-model="msg" @keyup.enter="sendMsg" bottom-slots label="Message">
						<template v-slot:before>
							<q-avatar>
								<img draggable="false" :src="'https://eu.ui-avatars.com/api/?name='+username">
							</q-avatar>
						</template>
						<template v-slot:append>
							<q-icon name="clear" class="cursor-pointer" @click="msg = ''" />
						</template>

						<template v-slot:after>
							<q-btn color="white" round dense flat icon="send" @click="sendMsg" />
						</template>
						</q-input>
					</div>
				</div>
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
			messages:[],
			img:'',
			isCall: false,
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
		this.getRandomString()
    
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
			this.$refs.mess.focus()
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
			getRandomString() {
				const randomChars =
					'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789'
				let result = ''
				for (let i = 0; i < 5; i++) {
					result += randomChars.charAt(
						Math.floor(Math.random() * randomChars.length),
					)
				}
				this.room = result
    },
  },
}
</script>

<style>

.connexion{
	margin: auto;
	background-color: #343434;
	padding: 25px;
}

.video-item{
	background-color: #2C3338 !important;
	border-radius: 10px !important;
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

/*
* Prefixed by https://autoprefixer.github.io
* PostCSS: v7.0.29,
* Autoprefixer: v9.7.6
* Browsers: last 4 version
*/

@use postcss-preset-env {
  stage: 0;
}

/* config.css */

:root {
  --baseColor: #606468;
}

/* helpers/align.css */

.align {
  display: -ms-grid;
  display: grid;
  place-items: center;
}

.grid {
  inline-size: 90%;
  margin-inline: auto;
  max-inline-size: 20rem;
}

/* helpers/hidden.css */

.hidden {
  border: 0;
  clip: rect(0 0 0 0);
  height: 1px;
  margin: -1px;
  overflow: hidden;
  padding: 0;
  position: absolute;
  width: 1px;
}

/* helpers/icon.css */

:root {
  --iconFill: var(--baseColor);
}

.icons {
  display: none;
}

.icon {
  block-size: 1em;
  display: inline-block;
  fill: var(--iconFill);
  inline-size: 1em;
  vertical-align: middle;
}

/* layout/base.css */

:root {
  --htmlFontSize: 100%;

  --bodyBackgroundColor: #2c3338;
  --bodyColor: var(--baseColor);
  --bodyFontFamily: "Open Sans";
  --bodyFontFamilyFallback: sans-serif;
  --bodyFontSize: 0.875rem;
  --bodyFontWeight: 400;
  --bodyLineHeight: 1.5;
}

* {
  -webkit-box-sizing: inherit;
          box-sizing: inherit;
}

html {
  -webkit-box-sizing: border-box;
          box-sizing: border-box;
  font-size: var(--htmlFontSize);
}

body {
  background-color: var(--bodyBackgroundColor);
  color: var(--bodyColor);
  font-family: var(--bodyFontFamily), var(--bodyFontFamilyFallback);
  font-size: var(--bodyFontSize);
  font-weight: var(--bodyFontWeight);
  line-height: var(--bodyLineHeight);
  margin: 0;
  min-block-size: 100vh;
}

/* modules/anchor.css */

:root {
  --anchorColor: #eee;
}

a {
  color: var(--anchorColor);
  outline: 0;
  text-decoration: none;
}

a:focus,
a:hover {
  text-decoration: underline;
}

/* modules/form.css */

:root {
  --formGap: 0.875rem;
}

input {
  background-image: none;
  border: 0;
  color: inherit;
  font: inherit;
  margin: 0;
  outline: 0;
  padding: 0;
  -webkit-transition: background-color 0.3s;
  -o-transition: background-color 0.3s;
  transition: background-color 0.3s;
}

input[type="submit"] {
  cursor: pointer;
}

.form {
  display: -ms-grid;
  display: grid;
  gap: var(--formGap);
}

.form input[type="password"],
.form input[type="text"],
.form input[type="submit"] {
  inline-size: 100%;
}

.form__field {
  display: -webkit-box;
  display: -ms-flexbox;
  display: flex;
}

.form__input {
  -webkit-box-flex: 1;
      -ms-flex: 1;
          flex: 1;
}

/* modules/login.css */

:root {
  --loginBorderRadus: 0.25rem;
  --loginColor: #eee;

  --loginInputBackgroundColor: #3b4148;
  --loginInputHoverBackgroundColor: #434a52;

  --loginLabelBackgroundColor: #363b41;

  --loginSubmitBackgroundColor: #ea4c88;
  --loginSubmitColor: #eee;
  --loginSubmitHoverBackgroundColor: #d44179;
}

.login {
  color: var(--loginColor);
}

.login label,
.login input[type="text"],
.login input[type="password"],
.login input[type="submit"] {
  border-radius: var(--loginBorderRadus);
  padding: 1rem;
}

.login label {
  background-color: var(--loginLabelBackgroundColor);
  border-bottom-right-radius: 0;
  border-top-right-radius: 0;
  padding-inline: 1.25rem;
}

.login input[type="password"],
.login input[type="text"] {
  background-color: var(--loginInputBackgroundColor);
  border-bottom-left-radius: 0;
  border-top-left-radius: 0;
}

.login input[type="password"]:focus,
.login input[type="password"]:hover,
.login input[type="text"]:focus,
.login input[type="text"]:hover {
  background-color: var(--loginInputHoverBackgroundColor);
}

.login input[type="submit"] {
  background-color: var(--loginSubmitBackgroundColor);
  color: var(--loginSubmitColor);
  font-weight: 700;
  text-transform: uppercase;
}

.login input[type="submit"]:focus,
.login input[type="submit"]:hover {
  background-color: var(--loginSubmitHoverBackgroundColor);
}

/* modules/text.css */

p {
  margin-block: 1.5rem;
}

.text--center {
  text-align: center;
}


</style>
