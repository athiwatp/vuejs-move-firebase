<template>

  <div draggable="true" class="avatar" :style="{'background': avatar.color, 'top': avatar.y + 'px', 'left': avatar.x + 'px'}" v-for="avatar in avatars"></div>
  <div class="bg" @dragover.prevent @drop="drop"></div>

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
    })
    Avatars.on('child_changed', function (snapshot) {
      var id = snapshot.key
      var avatar = vm.avatars.find(avatar => avatar.id === id)
      avatar.x = snapshot.val().x
      avatar.y = snapshot.val().y
      if (vm.myAvatar.id === id) {
        vm.myAvatar.x = snapshot.val().x
        vm.myAvatar.y = snapshot.val().y
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
    let y = 0

    return {
      avatars: [],
      myAvatar: {
        color: `rgb(${r}, ${g}, ${b})`,
        x,
        y
      }
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
    }
  }
}
</script>

<style>
.area {
  position: absolute;
  width: 100px;
  height: 100px;
  top:100px;
  right: 100px;
  background: #F00;
  border-color: red;
}
.avatar {
  position: absolute;
  width: 10px;
  height: 10px;
  background: #F00;
}
.bg {
  background:url(
            data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAoAAAAKCAYAAACNMs+9AAAAJklEQVQYV2NkAILQ6/+NV2synsVHM8IkQRrwAUZCCmDyoybiDSkAi64maRzs6ekAAAAASUVORK5CYII=
        ) repeat;
        width: 1280px;
        height: 700px;
}
</style>
