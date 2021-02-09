<template>
  <div id="geo-container">
    <svg></svg>
    <button>back</button>
  </div>
</template>

<script>
import { select, geoMercator, geoPath, zoom } from 'd3';
export default {
  data(){
    return {
      svg: null,
      xScale: null,
      selectedCounty:null,
      width:600,
      height:600,
      transitionDuration: 500,
      path:null,
      g:null, 
      svg:null,
      
    }
  },
  watch:{
    selectedCounty:function(newV, oldV){
      console.log(newV, oldV)
    }
  },
  methods:{
    selectMap:function(geojson,location){
      return geojson.filter( geoData => geoData.properties.county_id == location )
    },
    zzoom: function(){
      d3.zoom().scaleExtent([1, 2]).on('zoom', () => {
        g.style("stroke-width", 1 / d3.event.transform.k + "px");
        g.attr("transform", d3.event.transform); // updated for d3 v4
      })
    } 
  },
  mounted(){
    //Zoom
    let zzoom = d3.zoom().scaleExtent([1, 2]).on("zoom", zoomed);
    function zoomed() {
      g.style("stroke-width", 1 / d3.event.transform.k + "px");
      g.attr("transform", d3.event.transform); // updated for d3 v4
    }
    //makemap

    const makemap = (geojson) => {
      let path = g.selectAll('path').data(geojson)
      .enter().append('path')
      .attr('d',pathGenerator)
      .attr('class','boundary')
      .attr("stroke-width", 0.2)
      .attr("stroke", "#000000")
      .attr("fill", "#ffffff")
      
 
      path
      .on('mouseover', (d) => {
        d3.select(d)
        .transition()
        .duration(500)
        .attr("fill", "#DEB887")
        .attr('stroke','gray')
        .attr('stroke-opacity', 0.5)
        .attr("stroke-width", 1)
        
      })
      .on('mouseout', (d) => {
        d3.select(event.currentTarget)
        .transition()
        .duration(500)
        .attr("fill", "white")
        .attr('stroke','black')
        .attr('stroke-opacity', 1)
        .attr("stroke-width", 0.2)
      })
      .append('title')
      .text( d => d.properties.county)

      


      path.on('click',clicked)

      path.nodes().forEach( (d,i) => {
        setTimeout(()=>{
          let p = d3.select(d)
          repeat(p)
        },i * 100)
      });
      
      let repeat = (p) => {
        p
        .transition().duration(500).attr('fill', 'green').attr('fill-opacity',0.3)
        .transition().duration(500).attr('fill', 'white')
        .transition().duration(500).attr('fill', 'green').attr('fill-opacity',0.3)
        .transition().duration(500).attr('fill', 'white')
        .transition().duration(500).attr('fill', 'green').attr('fill-opacity',0.3)
        .transition().duration(500).attr('fill', 'white')
        .transition().duration(500).attr('fill', 'green').attr('fill-opacity',1)
      }

    }

    //zoomToBoundingBox
    const zoomToBoundingBox = d => {
      let bounds = pathGenerator.bounds(d),
          dx = bounds[1][0] - bounds[0][0],
          dy = bounds[1][1] - bounds[0][1],
          x = (bounds[0][0] + bounds[1][0]) / 2,
          y = (bounds[0][1] + bounds[1][1]) / 2,
        scale = Math.max(1, Math.min(10, .81/ Math.max(dx / this.width, dy / this.height))),
        translate = [this.width / 2 - scale * x, this.height / 2 - scale * y];
        svg.transition().duration(this.transitionDuration).call(
        zzoom.transform, d3.zoomIdentity.translate(translate[0],translate[1]).scale(scale)
      ); 
    }

    //clicked
    const clicked = d =>{
      d3.json('twTown.json').then(projectGeoJSON =>{
      // d3.json('project.json').then(projectGeoJSON =>{
      let projectgeojson = projectGeoJSON.features;
        console.log(d);
        zoomToBoundingBox(d);
        let selectedBlock = d.properties.county_id;
        this.selectedCounty = selectedBlock
        // let selectedBlock = d.properties.Block;
        let selectedjson = this.selectMap(projectgeojson,selectedBlock);
        g.selectAll("*").remove();
        makemap(selectedjson)
      })
    }


    d3.json('twCountry.json').then(json =>{
      makemap(json.features)
      d3.select('button').on('click',function(){
        g.selectAll("*").remove();
        svg.transition().duration(500).call( zzoom.transform, d3.zoomIdentity );
        makemap(json.features);
      })
    })

    const svg = select('svg')
    const g = svg.append('g');

    svg.attr('width', this.width).attr('height', this.height)
    const mercator = geoMercator().scale(5000)
    .translate([this.width/2, this.height/2])
    .center([121,23.2]);
    const pathGenerator = geoPath().projection(mercator);
  },
  created(){

  }
}
</script>

<style>

</style>