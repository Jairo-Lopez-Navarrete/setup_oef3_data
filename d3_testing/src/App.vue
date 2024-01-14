<script setup>
import { onMounted } from 'vue';
import * as d3 from 'd3';
import {ref} from 'vue';

let movieData = ref([]);
let xScale;

onMounted(async () => {
  const data = await d3.csv('movies.csv', typeConversion);
  const filteredMovies = data.filter(r => {
    return r.budget > 0 && r.genre && r.revenue > 0
  });
  const finishedData = prepareBarChart(filteredMovies);
  console.log(finishedData);
  movieData.value = finishedData;

  const extents = d3.extent(finishedData, d => d.revenue);
  xScale = d3.scaleLinear()
      .domain([0, extents[1]])
      .range([0, 200]);
})

function typeConversion(row)
{
  return {
    genre: row.genre.toLowerCase(),
    budget: +row.budget,
    revenue: +row.revenue
  };
}

function prepareBarChart(d)
{
  const dataMap = d3.rollup(
      d,
      r => d3.sum(r, x => x.revenue),
      d => d.genre
  )

  const dataArray = Array.from(dataMap, d => ({ genre: d[0], revenue: d[1] }));
  return dataArray;
}

</script>

<template>
  <svg width=700 height=350>
   <g v-for="(m, index) in movieData">
     <g :transform="`translate(${index * 55}, ${305 - xScale(m.revenue)})`">
       <rect :width="20" :height=xScale(m.revenue) fill=orange />
       <text transform="rotate(10,0,20)" text-anchor="middle" fill="black">{{ m.genre }}</text>
     </g>
   </g>
  </svg>
</template>

<style scoped>

</style>
