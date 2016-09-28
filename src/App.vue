<template>

  <div 
  draggable="true" 
  class="avatar" 
  :style="{'background': avatar.color, 'top': avatar.y + 'px', 'left': avatar.x + 'px'}" 
  v-for="avatar in avatars">
   {{ avatar.text }}
  </div>
  
  <div 
  class="bg" 
  @dragover.prevent 
  @drop="drop"></div>

<div class="textArea">
      <input type="text" v-model="avatarText">
      <button @click="addText(avatarText)">Send</button>
</div>

</template>

<script>
import firebase from 'firebase'

var config = {
  apiKey: 'AIzaSyBUNZ9z0AF3_6NvJAjmWeaAu1L_WlQjuA0',
  authDomain: 'vue1234-f8cf6.firebaseapp.com',
  databaseURL: 'https://vue1234-f8cf6.firebaseio.com',
  storageBucket: 'vue1234-f8cf6.appspot.com',
  messagingSenderId: '415861277156'
}
firebase.initializeApp(config)

var Avatars = firebase.database().ref('avatars')

export default {
  ready () {
    let vm = this
    vm.addAvatar(vm.myAvatar)
    Avatars.on('child_added', function (snapshot) {
      var item = snapshot.val()
      item.id = snapshot.key
      vm.avatars.push(item)
      console.log(vm.avatars, 'userID')
    })
    Avatars.on('child_changed', function (snapshot) {
      var id = snapshot.key
      var avatar = vm.avatars.find(avatar => avatar.id === id)
      avatar.x = snapshot.val().x
      avatar.y = snapshot.val().y
      avatar.text = snapshot.val().text
      if (vm.myAvatar.id === id) {
        vm.myAvatar.x = snapshot.val().x
        vm.myAvatar.y = snapshot.val().y
        vm.myAvatar.text = snapshot.val().text
      }
    })
    Avatars.on('child_removed', function (snapshot) {
      var id = snapshot.key
      vm.avatars.$remove(vm.avatars.find(avatar => avatar.id === id))
    })
  },
  data () {
    let r = Math.floor(Math.random() * 255)
    let g = Math.floor(Math.random() * 255)
    let b = Math.floor(Math.random() * 255)

    let x = Math.floor(Math.random() * 500)
    let y = Math.floor(Math.random() * 500)

    return {
      avatars: [],
      myAvatar: {
        color: `rgb(${r}, ${g}, ${b})`,
        x,
        y,
        text: ''
      },
      avatarText: ''
    }
  },

  // computed property for form validation state
  computed: {
  },

  // methods
  methods: {
    addAvatar: function (newAvatar) {
      let result = Avatars.push(newAvatar)
      this.myAvatar.id = result.key
    },
    drop: function (e) {
      console.log('X: ' + e.x + ' Y:' + e.y)
      let vm = this
      firebase.database().ref('avatars/' + vm.myAvatar.id).update({
        x: e.x,
        y: e.y
      })
    },
    addText: function (myAvatarText) {
      console.log(myAvatarText)
      let vm = this
      firebase.database().ref('avatars/' + vm.myAvatar.id).update({
        text: myAvatarText
      })
    }
  }
}
</script>

<style>
.avatar {
  position: absolute;
  width: 100px;
  height: 20px;
}
.bg {
  background:url(
            data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAoAAAAKCAYAAACNMs+9AAAAJklEQVQYV2NkAILQ6/+NV2synsVHM8IkQRrwAUZCCmDyoybiDSkAi64maRzs6ekAAAAASUVORK5CYII=
        ) repeat;
        width: 1280px;
        height: 800px;
}
* { margin: 0; padding: 0; box-sizing: border-box; }
      body { font: 13px Helvetica, Arial; }
      .textArea { background: #000; padding: 3px; position: fixed; bottom: 0; width: 100%; }
      .textArea input { border: 0; padding: 10px; width: 90%; margin-right: .5%; }
      .textArea button { width: 9%; background: rgb(130, 224, 255); border: none; padding: 10px; }
/*input[type="text"] {
    border: 1px solid #ddd;
    font-size: 11px;
    width: 125px;
    text-transform: uppercase;
    margin-top: 20px;
    margin-left: -48px;
}*/
</style>