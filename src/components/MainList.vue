<template>
  <div>
    <ol>
      <VolumeEntry
        v-for="item in volumeList"
        v-bind:key="item.id"
        v-bind:item="item"
        @deleteFig="figureEdited"
        @editFig="figureEdited"
      />
    </ol>
    </div>
</template>

<script>
import VolumeEntry from "./VolumeEntry"

export default {
    props: ["volumeList"],
    components: {
        VolumeEntry
    },
    methods: {
      figureEdited(signal) {
        this.$emit("listChanged", signal)
      }
    }
}
</script>

<style>
ol {
  display: inline-block;
}
</style>