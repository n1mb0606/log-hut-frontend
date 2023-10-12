<script setup>
import { ref, onMounted, watch } from "vue";
import * as d3 from "d3"

const data = ref([])
const startTime = ref(null)
const endTime = ref(null)
const timeRange = ref("2")

// Draw the chart based on data
function drawChart() {
  // Set chart size
  const width = window.innerWidth;
  const height = 400;
  const marginTop = 20;
  const marginRight = 20;
  const marginBottom = 30;
  const marginLeft = 40;

  // Declare the x (horizontal position) scale.
  const x = d3.scaleUtc([new Date(startTime.value), new Date(endTime.value)], [marginLeft, width - marginRight])

  // Declare the y (vertical position) scale.
  const y = d3.scaleLinear([0, 1], [height - marginBottom, marginTop])

  const svg = d3.select("#chart").attr("width", width).attr("height", height)

  // Edit the y-axis.
  svg.select("#xAxis")
      .transition()
      .attr("transform", `translate(0,${height - marginBottom})`)
      .call(d3.axisBottom(x));

  // Edit the y-axis.
  svg.select("#yAxis")
      .attr("transform", `translate(${marginLeft},0)`)
      .call(d3.axisLeft(y));

  // Renew data
  var dataset = data.value

  // Make liner
  const line = d3.line(
      (d) => {
        return x(new Date(d.Timestamp * 1000))
      },
      (d) => {
        return y(d.Ping)
      })

  // Renew the chart
  svg.select("path")
      .transition()
      .attr("class", "line")
      .attr("d", line(dataset))
      .style("fill", "none")
      .style("stroke", "#9E9D24")
      .style("stroke-width", "3");
}

async function fetchInitData() {
  console.log('Fetching init data..')


  const res = await fetch('http://192.168.49.2:32000/api/v1/log')
  const log = await res.json()

  startTime.value = new Date(log.Timestamp * 1000)
  if (log.Timestamp === 0) {
    log.Timestamp = new Date()
    log.value.Ping = 0.0
  }
  data.value.push(log)
}

async function fetchData() {
  console.log('Fetching New Data..')

  const res = await fetch('http://192.168.49.2:32000/api/v1/log')
  const log = await res.json()

  if (log.Timestamp === 0) {
    log.Timestamp = new Date()
    log.value.Ping = 0.0
  }
  data.value.push(log)
  endTime.value = new Date(log.Timestamp * 1000)
}

onMounted (() => {
  fetchInitData()
  drawChart()

  const intervalId = setInterval(() => {
        fetchData()
        drawChart()
        console.log(data.value[data.value.length - 1])
      },
      1000
  )
})
</script>
<template>
  <v-radio-group v-model="timeRange" density="default" hide-details inline>
    <v-radio
        color="lime-darken-3"
        label="Past 1h"
        value="1"
    ></v-radio>
    <v-radio
        color="lime-darken-3"
        label="Past 15m"
        value="2"
    ></v-radio>
    <v-radio
        color="lime-darken-3"
        label="Past 1m"
        value="3"
    ></v-radio>
  </v-radio-group>
  <v-card class="chartWrapper">
    <svg id="chart">
      <path id="path"></path>
      <g id="xAxis"></g>
      <g id="yAxis"></g>
    </svg>
  </v-card>
</template>

<style scoped>
.chartWrapper {
  margin: 10px;
}

</style>