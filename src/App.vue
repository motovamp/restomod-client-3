<template>
  <div>
    <div class="work">
      <!-- <a style="color: white" href="#" @click="test">test</a> -->
      <div class="layer car-mode" :class="mode"></div>
      <!-- div class="layer check" v-if="checkEngine"></div>
      <div class="layer park" v-if="park"></div-->
      <div class="layer gearbox" :class="{parking: shift === 0, rear: shift === 1, neutral: shift === 2, drive: shift === 3, three: shift === 4}"></div>
      <div class="low-info gold turn" :class="{active: mode === 'front'}">TURN</div>
      <div class="low-info gold exhst"  :class="{active: mode === 'back'}">EXHST</div>
      <div class="low-info state turn-state">{{turn ? 'SEQUENCE' : 'NORMAL'}}</div>
      <div class="low-info state exhst-state">{{boost ? 'BOOST' : 'NORMAL'}}</div>
      <div class="door-info" v-if="checkEngine">DOOR</div>
      <div class="hood-info" v-if="park">HOOD</div>
      <div class="programming" v-if="mode === 'main'">HOLD <span class="gold">OK</span> TO SELECT</div>
      <!--div class="programming" v-if="mode === 'switch'">Press left or rigth button <br/>to select option</div-->
      <div class="programming select" v-if="mode === 'front'">TURN SIGNALS</div>
      <div class="programming select" v-if="mode === 'back'">EXHAUST</div>
      <div class="programming select" v-if="mode === 'turn-sel'">
        <div class="label">TURN SIGNALS</div>
        <div class="info">&nbsp;{{sturn ? 'SEQUENCE' : 'NORMAL'}}</div>
      </div>
      <div class="programming select" v-if="mode === 'exh-sel'">
        <div class="label">EXHAUST</div>
        <div class="info">&nbsp;{{sboost ? 'BOOST' : 'NORMAL'}}</div>
      </div>
      <div class="programming" v-if="mode === 'programming'">Set gearbox shift lever in position <br/><span class="gold">{{gears[shift]}}</span> and press OK button</div>
      <div class="brake-info" v-if="info && !shift">PRESS <span class="gold">BRAKE</span> TO START</div>
      <div class="warn-info" v-else-if="info"><span class="red">WARNING</span><br/><span class="middle">VEHICLE IS NOT IN PARKING</span></div>
      <div class="malf-info" v-if="malf"></div>
      <div class="cross layer" v-if="check">
        <div class="vertical"></div>
        <div class="horizontal"></div>
      </div>
    </div>
  <div class="layers" v-if="loading">
    <div class="logo" v-if="logo">
      <img src="./assets/img/hstl.png">
    </div>
    <div class="gif" v-if="gif">
      <img @load="gifStarted()" src="./assets/img/Start.gif">
    </div>
  </div>
  </div>
</template>

<script>


export default {
  name: 'App',
  data() {
    return {
      loading: true,

      boost: false,
      turn: true,
      sboost: false,
      sturn: true,
      check: false,
      checkEngine: true,
      park: true,
      malf: false,
      mode: 'main',
      cmode: 'def',
      shift: 0,
      gears: ['P', 'R', 'N', 'D', '3'],
      socket: {},
      okTimer: 0,
      timer: 0,
      info: true, // cостояние вывода надписи нажать педаль
      infoTimer: 0,
      gif: false,
      logo: false,
      dstate: false, // состояние дисплея
    }
  },
  created() {
    document.addEventListener('keydown', key => {
      //eslint-disable-next-line
      // console.log(key)
      switch(key.key) {
        case 'Enter': this.holdOk(); break
        case 'ArrowLeft': this.cLeft(); break
        case 'ArrowRight': this.cRight(); break
        case 'ArrowUp': this.cUp(); break
        case 'ArrowDown': this.cDown(); break
        case ' ': this.cOk(); break
        case 'd': this.dstate = !this.dstate; break
        case 'p': this.mode = 'programming'; break
        case 'Escape': this.mode = 'main'; break
      }
    })

    //eslint-disable-next-line
    this.socket = io()
    this.socket.on('command', command => {
      switch(command) {
        case 'A': this.cUp(); break
        case 'B': this.cOk(); break
        case 'C': this.cRight(); break
        case 'D': this.cLeft(); break
        case 'E': this.cDown(); break
        case 'P': this.mode = 'programming'; this.shift = 0; break
        case 'M': if(this.mode == 'main') this.holdOk(); break
        case 'Q': this.shift = 0; break
        case 'R': this.shift = 1; break
        case 'S': this.shift = 2; break
        case 'T': this.shift = 3; break
        case 'U': this.shift = 4; break      
        case 'I': 
          this.info = true
        break
        case 'O': 
          this.loading = true
          this.logo = false
          this.gif = false
        break      
        case 'W': 
          this.info = false
        break      
        case 'Start': 
          this.dstate = true
        break
        case 'WStart': 
          this.dstate = true
          this.info = false
        break
        case 'Off': 
          this.dstate = false
        break      
      }
      //eslint-disable-next-line
      console.log(command)
    })
    // this.socket.on('response', response => {
    //   alert(response)
    // })

    this.socket.on('check', check => {
      //eslint-disable-next-line
      // console.log('check', check)
      if(check.toString() === '0') {
        this.checkEngine = false
      } else {
        this.checkEngine = true
      }
    })    
    this.socket.on('brake', brake => {
      //eslint-disable-next-line
      // console.log('brake', brake)
      if(brake.toString() === '0') {
        this.park = false
      } else {
        this.park = true
      }
    })
    this.socket.on('malf', malf => {
      //eslint-disable-next-line
      // console.log('malf', malf)
      if(malf.toString() === '1') {
        this.malf = true
      } else {
        this.malf = false
      }
    })  
    
  },
  mounted() {
        

  // датчики
    this.socket.on('sconnect', res => {
      if(res == 'success') {
        // Выключаем буст при загрузке
        this.socket.emit('exhst', 'normal')
        setInterval(() => {
          this.socket.emit('read', 'read')
        }, 500)
      }
    })

    // this.mode = 'turn'
    // setTimeout(() => {
    //   this.mode = 'back'
    //   // setTimeout(() => this.mode = 'main', 500)
    //   this.holdTime()
    // }, 500)    
  },
  methods: {
    cUp() {
      switch(this.mode) {
        case 'turn-sel':
          this.sturn = !this.sturn
        break
        case 'exh-sel':
          this.sboost = !this.sboost
        break
      }
    },
    
    cDown() {
      switch(this.mode) {
        case 'turn-sel':
          this.sturn = !this.sturn
        break
        case 'exh-sel':
          this.sboost = !this.sboost
        break
      }
    },
    
    cLeft() {
      switch(this.mode) {
        case 'front': 
          this.mode = 'back'
        break
        case 'back': 
          this.mode = 'front'
        break
        case 'turn-sel':
          this.sturn = !this.sturn
        break
        case 'exh-sel':
          this.sboost = !this.sboost
        break
      }
    },
    
    cRight() {
      switch(this.mode) {
        case 'front': 
          this.mode = 'back'
        break
        case 'back': 
          this.mode = 'front'
        break
        case 'turn-sel':
          this.sturn = !this.sturn
        break
        case 'exh-sel':
          this.sboost = !this.sboost
        break
      }
    },

    holdOk() {
      this.mode = 'front'
    },
    
    cOk() {
      switch(this.mode) {
        case 'front':
          this.mode = 'turn-sel' 
        break
        case 'back': 
          this.mode = 'exh-sel' 
        break
        case 'turn-sel':
          this.turn = this.sturn
          this.mode = 'front'
        break
        case 'exh-sel':
          this.boost = this.sboost
          this.mode = 'back'
        break
        case 'programming':
          if(this.shift < 4) {
          this.shift ++
        } else {
          this.mode = 'main'
        }
        break
      }

      // if(this.mode == 'programming') {
        
      // } else {
      //   if(this.mode === 'front') {
      //     this.mode = 'turn-sel'
      //   }
      // }
    },

    holdTime() {
      if(this.timer) {
        clearTimeout(this.timer)
      }
      this.timer = setTimeout(() => this.mode = 'main', 7000)
    },

    gifStarted() {
      //eslint-disable-next-line
      // console.log('gif')
    },    
    test() {
      this.socket.emit('test', 'sdildfsjkdfgjkldfgjklfdsjk')
    }
  },
  watch: {
    boost(value) {
      if(this.mode !== 'programming') this.holdTime()
      if(value) {
        this.socket.emit('exhst', 'boost')
      } else {
        this.socket.emit('exhst', 'normal')
      }
    },
    turn(value) {
      if(this.mode !== 'programming') this.holdTime()
      if(value) {
        this.socket.emit('turn', 'blink')
      } else {
        this.socket.emit('turn', 'normal')
      }
    },
    mode() {
      console.log(this.mode)
      if(this.mode !== 'programming') this.holdTime()
    },

    sturn() {
      if(this.mode !== 'programming') this.holdTime()
    },
    sboost() {
      if(this.mode !== 'programming') this.holdTime()
    },
    dstate(val) {
      if(val) {
        this.logo = true
        setTimeout(() => {
          this.logo = false
          this.gif = true
          setTimeout(() => {
            this.gif = false
            this.loading = false
          }, 2000)
        }, 1000)
      } else {
        this.loading = true
      } 
    }
  }
  
}
</script>

<style>
@import 'assets/css/main.css';
</style>
