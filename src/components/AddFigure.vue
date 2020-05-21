<template>
  <div class="figure-input">
    <div class="form-input">
      <div>
        <label for="param_figure1">{{figureList[figureType].param1}}:</label>
        <input type="text" id="param_figure1" v-model.lazy="val1" pattern="-?\d+[,\.]?\d*">
      </div>
      <div v-show="showInput2">
        <label for="param_figure2">{{figureList[figureType].param2}}:</label>
        <input type="text" id="param_figure2" v-model.lazy="val2" pattern="-?\d+[,\.]?\d*">
      </div>
      <div v-show="showInput3">
        <label for="param_figure3">{{figureList[figureType].param3}}:</label>
        <input type="text" id="param_figure3" v-model.lazy="val3" pattern="-?\d+[,\.]?\d*">
      </div>
      <div>
        <label for="factor_figure">Коэффициент:</label>
        <input type="text" id="factor_figure" v-model.lazy="factor" pattern="-?\d+[,\.]?\d*">
      </div>
      <div v-show="showCheckBox">
        <label for="checkBox1">{{figureList[figureType].checkBox1}}:</label>
        <input type="checkbox" 
        @change="figureList[figureType].checkAction" 
        v-model="checkState1"
        id="checkBox1">
      </div>
   </div>
    <div class="add-button">
      <label for="figure_select">Выбор фигуры:</label>
      <span class = "select">
        <select v-if="showSelect" id="figure_select" @change="setFigure">
          <option value="cylinder">Цилиндр</option>
          <option value="ball">Шар</option>
          <option value="cube">Куб</option>
          <option value="parallelepiped">Параллелипипед</option>
          <option value="hexprism">Призма (N=6)</option>
        </select>
    </span>
      <div> </div>
      <button v-on:click="addFigure">{{buttonName}}</button>
    </div>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        figureList: {
          cylinder: {param1:"Высота", param2:"Диаметр", calc:this.calcCyl},
          ball: {param1:"Диаметр", calc:this.calcBall},
          cube: {param1:"Сторона", calc:this.calcCube},
          parallelepiped: {param1:"Сторона 1", param2:"Сторона 2",
            param3:"Сторона 3", calc:this.calcParal},
          hexprism: {param1:"Диаметр вписаной окружности",
            param2:"Высота", calc:this.calcHex, checkAction: this.actionHex,
            checkBox1:"Описанная окружность"},
        },
        figureType: "cylinder",
        val1: "",
        showInput2: true,
        val2: "",
        showInput3: false,
        val3: "",
        factor: '1',
        showCheckBox: false,
        checkState1: false,
        showSelect: true,
        buttonName: "Добавить фигуру"
      }
    },
    props: {
      inital: {
        default: function() { 
        return {}
        }
      }
    },
    methods: {
      setFigure(e){
        this.figureType = e.target.value;
        this.initFigure()
      },
      initFigure(){
        if ("checkBox1" in this.figureList[this.figureType]) {
          this.showCheckBox = true;
        } else {
          this.showCheckBox = false;
        }
        if ('param2' in this.figureList[this.figureType]){
          this.showInput2 = true;
        } else {
          this.showInput2 = false;
          this.showInput3 = false;
          return
        }
        if ('param3' in this.figureList[this.figureType]){
          this.showInput3 = true;
        } else {
          this.showInput3 = false;
        }
      },
      addFigure() {
        let a = this.figureList[this.figureType].calc();
        if (!a.volume) return;
        this.$emit('newFigure', a);
      },
      getValues () {
        let p1, k;
        //преобразовать значения если используется запятая как десятичный разделитель
        if (this.val1.includes(',')) {
          p1 = +this.val1.replace(',','.');
        } else p1 = +this.val1;
        if (this.factor.includes(',')) {
          k = +this.factor.replace(',','.');
        } else k = +this.factor;
        if (this.showInput2) {
          let p2;
          if (this.val2.includes(',')) {
            p2 = +this.val2.replace(',','.');
          } else p2 = +this.val2;
          if (this.showInput3) {
            let p3;
            if (this.val3.includes(',')) {
              p3 = +this.val3.replace(',','.');
            } else p3 = +this.val3;
            //предпологается, что если хотя бы один параметр фигуры отрицальный, то её объём имеет отрицательное значение
            if (p1 < 0||p2 < 0||p3 < 0) {
              p1 = Math.abs(p1);
              p2 = Math.abs(p2);
              p3 = Math.abs(p3);
              if (k > 0) k = -k;
            }
            return [p1, p2, p3, k]
          } else {
              if (p1 < 0||p2 < 0) {
                p1 = Math.abs(p1);
                p2 = Math.abs(p2);
                if (k > 0) k = -k;
              }
              return [p1, p2, k]
            }
        } else {
          if (p1 < 0 && k > 1) p1 = -p1, k = -k;
          return [p1, k]
        }
      },
      calcCyl() {
        let [h, d, k] = this.getValues();
        let v = k*h*d**2*Math.PI/4000000;
        return {name:"Цилиндр", type:this.figureType,
        param1:this.figureList[this.figureType].param1,
        v1:this.val1, v2:this.val2,
        param2:this.figureList[this.figureType].param2,  
        k:this.factor, volume:v}
      },
      calcBall() {
        let [d, k] = this.getValues();
        let v = k*Math.PI*d**3/6000000;
        return {name:"Шар", type:this.figureType, param1:this.figureList[this.figureType].param1, 
        v1:this.val1, k:this.factor, volume:v}
      },
      calcCube() {
        let [a, k] = this.getValues();
        let v = k*a**3/1000000;
        return {name:"Куб", type:this.figureType, param1:this.figureList[this.figureType].param1, 
        v1:this.val1, k:this.factor, volume:v}
      },
      calcParal() {
        let [a, b, c, k] = this.getValues();
        let v = k*a*b*c/1000000;
        return {name:"Параллелипипед", type:this.figureType,
        param1:this.figureList[this.figureType].param1,
        param2:this.figureList[this.figureType].param2,
        param3:this.figureList[this.figureType].param3,         
        v1:this.val1, v2:this.val2, v3:this.val3,
        k:this.factor, volume:v}
      },
      calcHex() {
        let [d, h, k] = this.getValues();
        let v = this.checkState1 ? k*3*d**2*h*Math.sqrt(3)/8000000 : k*d**2*h*Math.sqrt(3)/2000000;
        return {name:"Шестигранная призма", param1:this.figureList[this.figureType].param1, 
        v1:this.val1, param2:this.figureList[this.figureType].param2, checkState1: this.checkState1,
        v2:this.val2, k:this.factor, volume:v}
      },
      actionHex() {
        if(this.checkState1) {
          this.figureList[this.figureType].param1 = "Диаметр описаной окружности";
        } else {
          this.figureList[this.figureType].param1 = "Диаметр вписаной окружности";
        }
      }
    },
    created() {
      if (!("type" in this.inital)) return
      this.showSelect = false;
      this.buttonName = "Изменить"
      if (!(this.inital.type in this.figureList)) return;
      ({type: this.figureType="cylinder", v1: this.val1=0, v2: this.val2=0, 
      v3: this.val3=0, k: this.factor='1', checkState1: this.checkState1=false} = this.inital);
      this.initFigure();
    }
  }
</script>

<style>
.figure-input {
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  padding-top: 0.5rem;
}

.figure-input .add-button {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  justify-content: center;
  padding-left: 0.5rem;
}

.figure-input .form-input {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  justify-content: space-between;
}

.figure-input input[type='text'] {
  border: none;
  padding: 0.5rem;
  font-size: 1.6rem;
  border-bottom: 0.1rem solid #bdbdbd;
  outline: none;
  width: 5em;
  background-color: inherit;
}

.figure-input label {
  color: #bdbdbd;
}

.figure-input input[type='checkbox'] {
  --active: #275EFE;
  -webkit-appearance: none;
  -moz-appearance: none;
  height: 1.6rem;
  width: 1.6rem;
  border-radius: 7px;
  outline: none;
  display: inline-block;
  vertical-align: middle;
  position: relative;
  margin: 0.5rem;
  cursor: pointer;
  border: 1px solid var(--bc, #BBC1E1);
  background: var(--b,  #fff);
  transition: background .3s, border-color .3s, box-shadow .2s;
}
input[type='checkbox']:after{
  content: '';
  display: block;
  left: 0;
  top: 0;
  position: absolute;
  transition: transform var(--d-t, 0.3s) var(--d-t-e, ease), opacity var(--d-o, 0.2s);
  opacity: var(--o, 0);
  width: 6px;
  height: 12px;
  border: 2px solid #fff;
  border-top: 0;
  border-left: 0;
  left: 7px;
  top: 4px;
  transform: rotate(var(--r, 20deg));
}
input[type='checkbox']:checked{
  --b: var(--active);
  --bc: var(--active);
  --d-o: .3s;
  --d-t: .6s;
  --d-t-e: cubic-bezier(.2, .85, .32, 1.2);
  --o: 1;
  --r: 43deg;
}
input[type='checkbox']:hover:not(:checked){
  --bc:  #275EFE;
}
input[type='checkbox']:focus{
  box-shadow: 0 0 0 2px rgba(39, 94, 254, .3);
}
</style>