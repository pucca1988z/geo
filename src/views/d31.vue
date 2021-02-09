<template>
  <div class="canvas"></div>
</template>

<script>
import { select, geoMercator, geoPath, zoom } from 'd3';
export default {
  data(){
    return{
      width:100,
      height:100,
      data:[1,2,3]
    }
  },
  mounted(){
    let svg = d3.select('.canvas').append('svg')
    svg
    .attr('width', 500)
    .attr('height', 250)

    
  }
}
</script>

<style>

</style>