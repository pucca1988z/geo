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
      dataset: [10, 20, 30, 40, 33, 24, 12, 5],
      width: 400,
      height: 400, 
      svg: null,
      xScale: null,
      padding: {left:30, right:30, top:20, bottom:20},
      selectedCounty:null
    }
  },
  watch:{
    selectedCounty:function(newV, oldV){
      console.log(newV, oldV)
    }
  },
  methods:{

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
      let path = g.on('click',()=>{
        console.log('mmm')
      }).selectAll('path').data(geojson)
      .enter().append('path')
      .attr('d',pathGenerator)
      .attr('class','boundary')
      .attr("stroke-width", 0.2)
      .attr("stroke", "#000000")
      .attr("fill", "#ffffff")

      //zoomToBoundingBox
      const zoomToBoundingBox = d => {
      let bounds = pathGenerator.bounds(d),
          dx = bounds[1][0] - bounds[0][0],
          dy = bounds[1][1] - bounds[0][1],
          x = (bounds[0][0] + bounds[1][0]) / 2,
          y = (bounds[0][1] + bounds[1][1]) / 2,
          scale = Math.max(1, Math.min(10, .81/ Math.max(dx / width, dy / height))),
          translate = [width / 2 - scale * x, height / 2 - scale * y];
          svg.transition().duration(transitionDuration).call(
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
          let selectedjson = selectMap(projectgeojson,selectedBlock);
          g.selectAll("*").remove();
          makemap(selectedjson)
        })
      }
      path.on('click',clicked)
    }

    //selectmap
    const selectMap = (geojson,location) => {
      let selection = [];
      geojson.forEach( sel =>{
        if (sel.properties.county_id == location){
        // if (sel.properties.Block == location){
          selection.push(sel);
        }
      });
      return selection;
    } 

    d3.json('twCountry.json').then(json =>{
    // d3.json('block.json').then(json =>{
      makemap(json.features)
      d3.select('button').on('click',function(){
        g.selectAll("*").remove();
        svg.transition().duration(transitionDuration).call( zzoom.transform, d3.zoomIdentity );
        makemap(json.features);
      })
    })

    const readGeoJSON = (filename) => {
      let a;
      let json = d3.json(`${filename}`).then(json =>{
        a=  json.features;
      })
      console.log(a);
    }
    let features = readGeoJSON('twTown.json');
    // let features = readGeoJSON('project.json');


    const svg = select('svg')
    const transitionDuration = 500;
    const width = document.body.clientWidth;
    const height = document.body.clientHeight;
    let active = d3.select(null);
    svg.attr('width', width).attr('height', height).on('click', ()=>{
      this.selectedCounty = null
    })
    const g = svg.append('g');
    const mercator = geoMercator().scale(5000)
    // const mercator = geoMercator().scale(10000)
    .translate([width/2, height/2])
    .center([121,23.2]);
    // .center([73,19.7])
    // const projection = d3.geoMercator()
    //   .fitExtent([[x0, y0], [x1, y1]],BeijingGeoJson) 
    const pathGenerator = geoPath().projection(mercator);
  },
  created(){

  }
}
</script>

<style>

</style>