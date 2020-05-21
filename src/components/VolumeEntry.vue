<template>
    <li class="entry-figure">
      {{item.name}}
      {{item | printParams}}
      Объём:{{item.volume | normDigits}}&nbsp;дм<sup>3</sup> 
      <a href="#" @click="deleteEntry">Удалить</a> 
      <a href="#" @click="showEdit = !showEdit">Изменить</a>
      <AddFigure v-if="showEdit"
      @newFigure="editEntry"
      v-bind:inital ="item" />
    </li>

</template>

<script>
import AddFigure from './AddFigure.vue';

export default {
  data() {
    return {
      showEdit: false
    }
  },
  props: { item : {
      required: true
    }
  },
  methods: {
    editEntry(fig) {
      fig.id = this.item.id;
      this.$emit("editFig", fig);
      this.showEdit = false;
    },
    deleteEntry(){
      this.$emit("deleteFig", this.item.id)
    }
  },
  filters: {
    normDigits(d) {
      //округляется до 6-го разряда, преобразуются в строку,
      //отбрасываются незначащие нули, заменяется точка на запятую 
      return d.toFixed(6)
              .replace(/^(-?\d+)\.(\d*?)0*$/,(s, s1, s2) => 
                s2=="" ? s1 : s1+','+s2
              );
    },
    printParams(d) {
      const s1 = d.param1+": "+d.v1
      if ('param2' in d) {
        const s2 = d.param2+": "+d.v2
        if ('param3' in d) return s1+"; "+s2+"; "+d.param3+": "+d.v3;
        return s1+"; "+s2;
      }
      return s1;
    }
  },
  components: {
    AddFigure
  }
}
</script>

<style>
.entry-figure {
  padding: 0.2rem;
  text-align: left;
}

.entry-figure:nth-child(odd) {
  background-color: rgb(228, 232, 235);
}

.entry-figure a {
  border-bottom: 2px dashed #BBC1E1;
  text-decoration: none;
  color: #777;
  margin: 0 1rem;
}
.entry-figure a:hover {
    border-bottom-style: dotted;
}

.entry-figure a:active {
    border-bottom-style: solid;
}

</style>