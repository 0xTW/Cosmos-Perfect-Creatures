<template>
  <!-- <img alt="Vue logo" src="./assets/logo.png"> -->
  <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->

  <div class="container-container container-lg">
    <h1 class="text-danger">Distributed Power Generation Solution of Venus Exploration 🌌</h1>

    <div v-if="start === false">
      <div class="fs-2 mb-3">
        Please choose your machine type (solution):
      </div>

      <div class="btn-group-vertical" role="group" aria-label="Vertical radio toggle button group">
        <input type="radio" class="btn-check" name="vbtn-radio" id="vbtn-radio1" autocomplete="off" v-model="solution" value=1>
        <label class="btn btn-outline-primary fs-4 text-start" for="vbtn-radio1">Traditional Solution</label>
        <input type="radio" class="btn-check" name="vbtn-radio" id="vbtn-radio2" autocomplete="off" v-model="solution" value=2>
        <label class="btn btn-outline-primary fs-4 text-start" for="vbtn-radio2">Distributed Power Generation Solution(Spring)</label>
        <input type="radio" class="btn-check" name="vbtn-radio" id="vbtn-radio3" autocomplete="off" v-model="solution" value=3>
        <label class="btn btn-outline-primary fs-4 text-start" for="vbtn-radio3">Distributed Power Generation Solution(Compressed Air)</label>
      </div>

      <div class="my-3" v-if="solution > 0">
        <div class="btn btn-lg btn-primary" @click="startSimulator()">START 🚀</div>
      </div>
    </div>

    <div v-if="start === true && endSimulator === false">
      <div class="fs-2 mb-3">
        Please choose your action:
      </div>
    </div>

    <div class="row my-3" v-if="start === true && endSimulator !== true">
      <div class="col-12 col-lg-4 mb-3 mb-lg-0 d-flex">
        <div class="card w-100">
          <div class="card-body d-flex flex-column justify-content-between">
            <h4 class="card-title fs-2">Explore</h4>
            <p class="card-text fs-5 text-start">Explore Venus, It will cost <span class="text-danger">{{ Math.max(eleCapacity - 20, 20) }}</span> Wh electricity.</p>
            <div class="btn btn-lg btn-outline-info w-50 mx-auto" :class="eleCapacity >= 20 ? '' : 'invisible'" @click="explore()">Execute</div>
          </div>
        </div>
      </div>
      <div class="col-12 col-lg-4 mb-3 mb-lg-0 d-flex" v-if="solution !== '1'">
        <div class="card w-100">
          <div class="card-body d-flex flex-column justify-content-between">
            <h4 class="card-title fs-2">Build Power Station</h4>
            <p v-if="eleStations < 3" class="card-text fs-5 text-start">Build new power station to get more battery per day, it will cost <span class="text-danger">{{ buildStationCost}}</span> Wh electricity to build.</p>
            <p v-if="eleStations == 3" class="card-text fs-5 text-start">The maximum of power stations is 3.</p>
            <div class="btn btn-lg btn-outline-info w-50 mx-auto" :class="(eleStations < 3 && eleCapacity >= buildStationCost) ? '' : 'invisible'" @click="buildPowerStation()">Execute</div>
          </div>
        </div>
      </div>
      <div class="col-12 col-lg-4 mb-3 mb-lg-0 d-flex">
        <div class="card w-100">
          <div class="card-body d-flex flex-column justify-content-between">
            <h4 class="card-title fs-2">Call it a day</h4>
            <p class="card-text fs-5 text-start">Wait for generate electricity to charge battery, and call it a day.</p>
            <div class="btn btn-lg btn-outline-info w-50 mx-auto" @click="callItADay()">Execute</div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="endSimulator === true" class="fs-1 border border-primary border-2 p-3">
      {{ message }} <br/>
      <div class="btn btn-lg btn-outline-dark" @click="restartSimulator()">Restart Simulator</div>
    </div>

    <div class="bg-dark rounded-3 p-3 text-white mt-4">
      <div class="d-flex justify-content-start">
        <img src="./assets/lightbulb.svg" style="height: 80px" class="me-3">
        <div class="fs-3 d-flex flex-column justify-content-center">
          <div>Solution: {{ solutionName }} <span v-if="start===true" class="btn btn-sm btn-outline-warning" @click="restartSimulator()">Restart Simulator</span></div>
        </div>
      </div>

      <div class="d-flex justify-content-start">
        <img src="./assets/calendar.svg" style="height: 80px" class="me-3">
        <div class="fs-3 d-flex flex-column justify-content-center"><div>Days: {{ days }}</div></div>
      </div>

      <div class="d-flex justify-content-start">
        <img v-if="(eleCapacity / maxCapacity) >= 0.9" src="./assets/battery-100.svg" style="height: 80px" class="me-3">
        <img v-if="(eleCapacity / maxCapacity) >= 0.55 && (eleCapacity / maxCapacity) < 0.9" src="./assets/battery-75.svg" style="height: 80px" class="me-3">
        <img v-if="(eleCapacity / maxCapacity) >= 0.15 && (eleCapacity / maxCapacity) < 0.55" src="./assets/battery-50.svg" style="height: 80px" class="me-3">
        <img v-if="(eleCapacity / maxCapacity) > 0 && (eleCapacity / maxCapacity) < 0.15" src="./assets/battery-25.svg" style="height: 80px" class="me-3">
        <img v-if="maxCapacity === 0 || ((eleCapacity / maxCapacity) <= 0.)" src="./assets/battery-0.svg" style="height: 80px" class="me-3">
        <div class="fs-3 d-flex flex-column justify-content-center">
            <div>Battery: {{ eleCapacity }} Wh
              <span v-if="days > 0" style="color: Lightgoldenrodyellow" class="fs-5 text-start">Note: previous day, generate <span style="color: gold">{{ previousGenerate }} Wh</span> electricity to charge battery.</span>
            </div>
        </div>
      </div>

      <div class="d-flex justify-content-start">
        <img src="./assets/compass.svg" style="height: 80px" class="me-3">
        <div class="fs-3 d-flex flex-column justify-content-center"><div>Explore: {{ Math.round(exploreProgress * 100) / 100}} %</div></div>
      </div>

      <div class="d-flex justify-content-start" v-if="eleStations > 0">
        <div class="fs-3 d-flex flex-column justify-content-center"><div>Power Stations:</div></div>
        <img v-bind:key="i" v-for="i in [...Array(eleStations).keys()]" src="./assets/wind-energy.svg" alt="Station" style="height: 80px">
      </div>
    </div>

    <div class="d-block d-lg-flex justify-content-start text-start my-3">
      <a class="btn btn-outline-info me-3 mb-2 mb-lg-0" href="https://github.com/0xTW/Cosmos-Perfect-Creatures">Github Project</a>
      <div class="d-flex justify-content-center flex-column">
        <div>
        All emojis(svg) designed by <a href="https://openmoji.org/" rel="nofollow" target="_blank">OpenMoji</a> – the open-source emoji and icon project. License: <a href="https://creativecommons.org/licenses/by-sa/4.0/#" rel="nofollow" target="_blank">CC BY-SA 4.0</a>.</div>
      </div>
    </div>

  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'

export default {
  name: 'App',
  data() {
    return {
      solution: 0,
      start: false,
      endSimulator: false,
      days: 1,
      exploreProgress: 0,
      eleStations: 0,
      eleCapacity: 0,
      maxCapacity: 0,
      previousGenerate: 0,
      buildStationCost: 0,
      message: ''
    }
  },
  methods:{
    startSimulator(){
      switch (this.solution) {
        case '1':
          this.eleCapacity = 1100
          this.maxCapacity = 1100
          this.buildStationCost = 0
          break;
        case '2':
          this.eleCapacity = 220
          this.maxCapacity = 220
          this.buildStationCost = 200
          break;
        case '3':
          this.eleCapacity = 440
          this.maxCapacity = 440
          this.buildStationCost = 150
          break;
        default:
          break;
      }
      this.start = true;
    },
    generateEle(){
        if (this.solution === '1')
        {
          return this.choice([0, 108, 216, 270])
        }
        else if (this.solution === '2')
        {
            let acc = 0;
            for (let index = 0; index < this.eleStations; index++) {
              acc += this.choice([0, 5, 10, 11])
            }
            return acc
        }
        else if (this.solution === '3')
        {
            let acc = 0;
            for (let index = 0; index < this.eleStations; index++) {
              acc += this.choice([0, 30, 60, 65])
            }
            return acc
        }
    },
    explore(){
      if (this.eleCapacity >= 20)
      {
          this.exploreProgress += (this.eleCapacity - 20) / 500
          this.eleCapacity = 20
          this.callItADay()
      }
      else
      {
        console.log('Electricity is not enough to explore.')
      }
    },
    buildPowerStation(){
        if (this.solution === '1')
        {
            console.log('Not thing happend.')
        }
        else if (this.eleStations >= 3)
        {
            console.log('The maximum of power stations is 3.')
        }
        else if (this.eleCapacity >= this.buildStationCost)
        {
          this.eleStations += 1
          this.eleCapacity -= this.buildStationCost
        }
        else
        {
          console.log('Electricity is not enough to build.')
        }
    },
    callItADay(){
      this.previousGenerate = this.generateEle();
      this.eleCapacity = Math.min((this.eleCapacity + this.previousGenerate), this.maxCapacity);
      this.eleCapacity -= 5
      this.days += 1;

      if (this.solution === '1' && this.eleCapacity < 0) {
        this.eleCapacity = 0
      }

      this.endSimulatorCheck()
    },
    endSimulatorCheck()
    {
      let flag = false;
      if (this.eleCapacity < 0 && this.solution !== '1'){
        this.message = 'Machine is out of electricity, the simulator is done.';
        flag = true;
      }
      if (this.exploreProgress >= 100){
        this.message = 'Venus Explore is success, Congrats 😉';
        flag = true;
      }
      if (this.days == 60){
        this.message = '60 days exploration in Venus is over.';
        flag = true;
      }

      if (flag === true) {
        this.endSimulator = true;
      }
    },
    restartSimulator(){
      this.solution = 0,
      this.start = false,
      this.endSimulator = false,
      this.days = 1,
      this.exploreProgress = 0,
      this.eleStations = 0,
      this.eleCapacity = 0,
      this.maxCapacity = 0,
      this.previousGenerate = 0,
      this.buildStationCost = 0,
      this.message = ''
    },
    choice(arr){
      return arr[Math.floor(Math.random() * arr.length)]
    }
  },
  computed: {
    solutionName(){
      if (this.solution === '1')
      {
        return 'Traditional';
      }
      else if (this.solution === '2')
      {
        return 'Distributed Power Generation Solution(Spring)';
      }
      else if (this.solution === '3')
      {
        return 'Distributed Power Generation Solution(Compressed Air)';
      }
      else{
        return ''
      }
    }
  },
  components: {
    // HelloWorld
  }
}
</script>

<style>
@import'~bootstrap/dist/css/bootstrap.css';
@import url('https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap');

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 20px;
}

.card:hover{
  border-color: dodgerblue;

}
</style>
