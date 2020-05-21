<template>
  <div>
    <h1>Общий объём: {{result | displayResult}} дм<sup>3</sup></h1>
    <h2 v-if="dencity!=0">Масса: {{mass}} грамм </h2>
  </div>
</template>

<script>
  export default {
    props: ["result", "dencity"],
    filters: {
      displayResult(d) {
        let exp;
        if (d==0) return d;
        if (Math.abs(d) > 1) {
          exp = 3
        } else {
          exp = -Math.trunc(Math.log10(Math.abs(d))-3);
        }
        return d.toFixed(exp)
                .replace(".",",");
      }
    },
    computed: {
      mass () {
        let m = this.result * this.dencity;
        return m.toFixed(2)
      }
    }
  }
</script>