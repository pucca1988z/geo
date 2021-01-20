<template>
  <div id="geo-container">
    Geographic
  </div>
</template>

<script>
import * as d3 from 'd3'
import country from "../assets/geoData/COUNTY_MOI_1090820.json"
export default {
  data(){
    return {
      svg: null,
      width: 975, height:610
    }
  },
  methods:{
    getCountryData(){
      return new Promise((resolve, reject) => {
        resolve(import("../assets/geoData/COUNTY_MOI_1090820.json"))
      }) 
    }
  },
  mounted(){
    this.svg = 
    d3.select('div#geo-container')
      .append('svg')
      .attr('cursor', 'pointer')
      // .attr("preserveAspectRatio", "xMinYMin meet")
      .style("background-color","#c9e8fd")
      .attr("viewBox", "0 0 " + this.width + " " + this.height)
      .classed("svg-content", true);

    // load data
    let country = this.getCountryData().then(res => d3.json(res.default))
    // country = country.default
    // console.log(country)
    // let url = '../assets/geoData/COUNTY_MOI_1090820.json'
    // let countryData = d3.json(url, (err, geometry) => {
    //   if(err) console.log(err)
    // })
  }
}
</script>

<style>

</style>