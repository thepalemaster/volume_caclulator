<template>
  <div id="app">
    <h2>Калькулятор расчёта объёма и массы по объёму.</h2>
    <p>Объём сложного объекта, как правило, можно представить как сумму (или разницу) простых фигур. <br>
    С помощью этого калькулятора таким образом можно рассчитать объём для сложной фигуры, а для некоторых материалов через удельную плотность вычислить массу фигуры.
    </p>
    <DisplayResult
      v-bind:result = "result"
      v-bind:dencity = "den"
    />
    <span class ="select">
      <label for="select-material">Материал объекта: </label>
      <select id="select-material" @change="changeDen">
        <option value="0">Выберете материал</option>
        <option value="7800">Сталь</option>
        <option value="2700">Алюминий</option>
        <option value="8920">Медь</option>
        <option value="-1">Особая плотность</option>
      </select>
    </span>
    <div v-if="showCustomDen">
      <label for="custom-den">Особая плотность: 
      <input class="special-dencity" id="custom-den" type="text" v-model="den">
      г/дм<sup>3</sup></label>
    </div>
    <AddFigure @newFigure="addEntry" />
    <button v-show="volumeList.length" @click="volumeList=[]">Сбросить</button>
    <MainList
    @listChanged = "updateList"
    v-bind:volumeList="volumeList" />
  </div>
</template>

<script>
import MainList from './components/MainList.vue';
import DisplayResult from './components/DisplayResult.vue';
import AddFigure from './components/AddFigure.vue';

export default {
  name: 'App',
  data() {
    return {
      volumeList: [],
      den: 0,
      volume: 100,
      figureID: 0,
      showCustomDen: false
    }
  },
  computed: {
    result () {
      return this.volumeList.reduce((summ, i) =>
        summ+=i.volume, 0
      )
    }
  },
  methods: {
    addEntry(fig) {
      this.figureID++;
      fig.id = this.figureID;
      this.volumeList.push(fig);
    },
    changeDen(e) {
      if (e.target.value < 0) {
        this.den = 0;
        this.showCustomDen = true;
      } else {
        this.den = +e.target.value
        if (this.showCustomDen) this.showCustomDen = false;
      }
    },
    updateList(signal) {
      if (Number.isInteger(signal)) {
        let i = this.getFigureByID(signal)
        this.volumeList.splice(i,1);
        return
      }
      if (typeof signal == "object") {
        let i = this.getFigureByID(signal.id);
        this.$set(this.volumeList, i, signal);
      }
    },
    getFigureByID(num){
      for (let i=0; i < this.volumeList.length; i++) {
        if (this.volumeList[i].id==num) return i;
      }
    }
  },
  components: {
    MainList, DisplayResult, AddFigure
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

button {
  font-family: inherit;
  margin: 15px 0;
  appearance: none;
  background: #BBC1E1;
  border-radius: 5px;
  border: 0;
  font-size: 1rem;
  padding: 0.5rem;
  cursor: pointer;
}

button:hover {
  background: #BBD9E4;
}

button:focus {
  outline: none;
  box-shadow: 0 0 0 2px #ccd6ee;
}

.select {
  position: relative;
  display: inline-block;
  overflow: hidden;
}

.select select {
  cursor:pointer;
  background-color: #BBC1E1;
  font-size: inherit;
  padding: .5em;
  padding-right: 2.5em;	
  border: 0;
  margin: 0;
  border-radius: 5px;
  -webkit-appearance: none;
  -moz-appearance: none;

}

.select::before,
.select::after {
  content: "";
  position: absolute;
  pointer-events: none;
}

.select::after { 
  content: "\25BC";
  line-height: 1;
  right: 0.5em;
  top: 0.5em;
}

.select::before { 
  width: 2em;
  right: 0;
  top: 0;
  bottom: 0;
  border-radius: 0 5px 5px 0;
}

.select::before {
  background-color: rgba(0,0,0,.15);
}

.special-dencity {
  background-color: #BBC1E1;
  font-size: inherit;
  width: 5em;
  border-radius: 5px;
  padding: .5em;
  border: none;
}

</style>
