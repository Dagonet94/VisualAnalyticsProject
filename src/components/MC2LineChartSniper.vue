<template>
  <div id="container">
    <div id="dataSelection">
      <div id="sliderWrapper">
        <div id="sliderContainer">
          <vue-slider
            v-model="sliderValue"
            :min="min"
            :max="max"
            :step="step"
            :width=1000
            :tooltip-formatter="formatter"
            :tooltip= "'always'"
            :lazy = "true"
            :enable-cross= "false"
          ></vue-slider>
        </div>
      </div>
      <div id="selectionWrapper">
        <div id="checkboxContainer">
          <p>Location</p>
          <!-- eslint-disable -->
          <div id="checkboxLocation">
            <label v-bind:class="[{ 'isActiveBoonsri': isActiveBoonsri }]">Boonsri</label>
            <input @click="isActiveBoonsri = !isActiveBoonsri"type="checkbox" value="Boonsri" v-model="locationArray">
            <label v-bind:class="[{ 'isActiveKannika': isActiveKannika }]" >Kannika</label>
            <input @click="isActiveKannika = !isActiveKannika" type="checkbox" value="Kannika" v-model="locationArray">
            <label v-bind:class="[{ 'isActiveChai': isActiveChai }]" >Chai</label>
            <input @click="isActiveChai = !isActiveChai" type="checkbox" value="Chai" v-model="locationArray">
            <label v-bind:class="[{ 'isActiveKohsoom': isActiveKohsoom }]">Kohsoom</label>
            <input @click="isActiveKohsoom = !isActiveKohsoom" type="checkbox" value="Kohsoom" v-model="locationArray">
            <label v-bind:class="[{ 'isActiveSomchair': isActiveSomchair }]">Somchair</label>
            <input @click="isActiveSomchair = !isActiveSomchair" type="checkbox" value="Somchair" v-model="locationArray">
            <label v-bind:class="[{ 'isActiveSakda': isActiveSakda }]">Sakda</label>
            <input @click="isActiveSakda = !isActiveSakda" type="checkbox" value="Sakda" v-model="locationArray">
            <label v-bind:class="[{ 'isActiveBusarakhan': isActiveBusarakhan }]">Busarakhan</label>
            <input @click="isActiveBusarakhan = !isActiveBusarakhan" type="checkbox" value="Busarakhan" v-model="locationArray">
            <label v-bind:class="[{ 'isActiveTansanee': isActiveTansanee }]">Tansanee</label>
            <input @click="isActiveTansanee = !isActiveTansanee" type="checkbox" value="Tansanee" v-model="locationArray">
            <label v-bind:class="[{ 'isActiveAchara': isActiveAchara }]">Achara</label>
            <input @click="isActiveAchara = !isActiveAchara" type="checkbox" value="Achara" v-model="locationArray">
            <label v-bind:class="[{ 'isActiveDecha': isActiveDecha }]">Decha</label>
            <input @click="isActiveDecha = !isActiveDecha" type="checkbox" value="Decha" v-model="locationArray">
          </div>
        </div>
        <div id="measureContainer">
          <label>Measure</label>
          <select v-model="measure">
            <!-- eslint-disable -->
            <option v-for='measures in sinMeasure'>{{measures}}</option>
          </select>
        </div>
      </div>
    </div>
    <div class="col-md-12">
      <div id="chart-area"></div>
    </div>

  </div>
</template>

<script>
/* eslint-disable */
  import VueSlider from 'vue-slider-component'
  import Vue from 'vue'
  import * as d3 from 'd3'
  import 'vue-slider-component/theme/default.css'
  //import legend from 'd3-svg-legend'
  // import jQuery from 'jQuery'
  Vue.component('VueSlider', VueSlider)
  // eslint-disable-next-line
let legend
  let timeArray
  let tempArray = []
  let scaleArray = []
  let line
  let svg
  let x
  let y
  let margin = { left: 80, right: 100, top: 50, bottom: 100 }
  let height = 500 - margin.top - margin.bottom
  let width = 800 - margin.left - margin.right
  let yAxis
  let xAxis
  let g
  let xLabel
  let yLabel
  let xAxisCall
  let yAxisCall
  let multydataFilteredArray = [[], [], [], [], [], [], [], [], [], []]
  let parseTime = d3.timeParse('%Y-%m-%d')
  let formatTime = d3.timeFormat('%d/%m/%Y')

  export default {
    name: 'MC2LineChartSniper',
    data () {
      return {
        unit: 'C',
        colorArray:[{'location':'Boonsri', 'color': 'purple'}, {'location':'Kannika', 'color': 'red'}, {'location':'Chai', 'color': 'blue'}, {'location':'Kohsoom', 'color': 'yellow'}, {'location':'Somchair', 'color': '#089514'}, {'location':'Sakda', 'color':'#43cbd5'}, {'location':'Busarakhan', 'color': '#c8c86a'}, {'location':'Tansanee', 'color': '#8d6695'}, {'location':'Achara', 'color': '#525c95'}, {'location':'Decha', 'color':'#95911c'}],
        isActiveBoonsri: true,
        isActiveKannika: false,
        isActiveChai: false,
        isActiveKohsoom:false,
        isActiveSomchair:false,
        isActiveSakda:false,
        isActiveBusarakhan:false,
        isActiveTansanee:false,
        isActiveAchara:false,
        isActiveDecha: false,
        formatter: v => `${formatTime((new Date(v)))}`,
        max: parseTime('2016-12-31').getTime(),
        min: parseTime('1998-01-11').getTime(),
        step: 86400000, // One day
        sliderValue: [parseTime('1998-01-11').getTime(), parseTime('2016-12-31').getTime()],
        locations: null,
        locationArray: ['Boonsri'],
        measureArray: ['Water temperature'],
        dataArray: this.readyArray,
        dataFilteredArray: [],
        sinLocation: this.singleLocation,
        sinMeasure: this.singleMeasure,
        location: 'Boonsri',
        measure: 'Water temperature'
        // maxDate: parseTime('1998-01-11').start(),
      }
    },
    components: {
      VueSlider
    },
    props: {
      readyArray: {
        type: Array,
        require: true
      },
      singleLocation: {
        type: Array,
        require: true
      },
      singleMeasure: {
        type: Array,
        require: true
      }
    },

    mounted () {
      tempArray = []
      scaleArray = []
      for (let i = 0; i < this.dataArray.length; i++) {
        if ((this.dataArray[i].measure === this.measure) && (typeof this.dataArray[i].value === 'number')) {
          scaleArray.push(this.dataArray[i])
          if (this.dataArray[i].location === this.location) {
            tempArray.push(this.dataArray[i])
          }
        }
      }
      this.dataFilteredArray = tempArray

      svg = d3.select('#chart-area')
        .append('svg')
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom)

      g = svg.append('g')
        .attr('transform', 'translate(' + margin.left +
          ', ' + margin.top + ')')

      xLabel = g.append('text')
        .attr('class', 'x axisLabel')
        .attr('y', height + 50)
        .attr('x', width / 2)
        .attr('font-size', '20px')
        .attr('text-anchor', 'middle')
        .text('Time')
      yLabel = g.append('text')
        .attr('class', 'y axisLabel')
        .attr('transform', 'rotate(-90)')
        .attr('y', -60)
        .attr('x', -170)
        .attr('font-size', '20px')
        .attr('text-anchor', 'middle')
        .text(this.measure)

      x = d3.scaleTime()
        .range([0, width])
        .domain(d3.extent(this.dataFilteredArray, function (d) {
          return d.sampleDate
        }))

      y = d3.scaleLinear()
        .range([height, 0])
        .domain([d3.min(scaleArray, function (d) {
          return d.value
        }),
          d3.max(scaleArray, function (d) {
            return d.value
          })])

      // Define the x axis
      xAxisCall = d3.axisBottom().scale(x)
      // Append asse x
      xAxis = g.append('g')
        .attr('class', 'x axis')
        .attr('transform', 'translate(0,' + height + ')')

      // define the yaxis
      yAxisCall = d3.axisLeft().scale(y)
      // Append asse y
      yAxis = g.append('g')
        .attr('class', 'y axis')

      // Prima linea
      g.append('path')
        .attr('class', 'line')
        .attr('fill', 'none')
        .attr('stroke', 'grey')
        .attr('stroke-width', '2px')
        .attr('id', 'Boonsri')

       legend= svg.append('g')
        .attr('class', 'legend')
        .attr('transform', 'translate('+ (width-100)+ ',' + 20 + ')')
        .selectAll('g')
        .data(['Test'])
        .enter().append('g')

      this.update()
    },

    methods: {
      // metodo di update del grafico
      update: function () {
        x = d3.scaleTime()
          .range([0, width])
          .domain(d3.extent(this.dataFilteredArray, function (d) {
            return d.sampleDate
          }))

        y = d3.scaleLinear()
          .range([height, 0])
          .domain([d3.min(scaleArray, function (d) {
            return d.value
          }),
            d3.max(scaleArray, function (d) {
              return d.value
            })])

        // domini delle scale
        xAxisCall.scale(x)
        xAxis.transition(this.transition()).call(xAxisCall)
        yAxisCall.scale(y)
        yAxis.transition(this.transition()).call(yAxisCall.tickFormat(d=>d + this.unit))

        line = d3.line()
          .x(function (d) {
            return x(d.sampleDate)
          })
          .y(function (d) {
            return y(d.value)
          })
        g.select('.line')
          .transition(this.transition())
          .attr('d', line(this.dataFilteredArray))
          .attr('stroke', 'purple')

      },

      multyUpdate: function () {
        x = d3.scaleTime()
          .range([0, width])
          .domain(d3.extent(timeArray, function (d) {
            return d.sampleDate
          }))

        y = d3.scaleLinear()
          .range([height, 0])
          .domain([d3.min(scaleArray, function (d) {
            return d.value
          }),
            d3.max(scaleArray, function (d) {
              return d.value
            })])

        // domini delle scale
        xAxisCall.scale(x)
        xAxis.transition(this.transition()).call(xAxisCall)
        yAxisCall.scale(y)
        yAxis.transition(this.transition()).call(yAxisCall.tickFormat(d=>d + this.unit))
        for (let i = 0; i < multydataFilteredArray[[i]].length; i++) {
          // eslint-disable-next-line eqeqeq
          if (multydataFilteredArray[[i]].length != 0) {
            let dataArray = multydataFilteredArray[i]
            let id = dataArray[0].location

            let color
            for (let l =0; l< this.colorArray.length; l++){
              if(this.colorArray[l].location===id)
              {
                console.log(this.colorArray[l].color)
                color= this.colorArray[l].color
              }
            }

            line = d3.line()
              .x(function (d) {
                return x(d.sampleDate)
              })
              .y(function (d) {
                return y(d.value)
              })
            g.append('path')
              .attr('class', 'line')
              .attr('id', id)
              .attr('fill', 'none')
              .attr('stroke', color)
              .attr('stroke-width', '2px')
              .attr('d', line(dataArray))
          }
        }
      },

      transition: function () {
        return d3.transition().duration(1000)
      }
    },

    watch: {
      locationArray () {
        tempArray = [[], [], [], [], [], [], [], [], [], []]
        scaleArray = []
        timeArray = []
        for (let m = 0; m < this.locationArray.length; m++) {
          this.location = this.locationArray[m]
          for (let i = 0; i < this.dataArray.length; i++) {
            if ((this.dataArray[i].measure === this.measure) && (typeof this.dataArray[i].value === 'number')) {
              scaleArray.push(this.dataArray[i])
              if ((this.dataArray[i].location === this.location) &&
                (this.dataArray[i].sampleDate.getTime() >= this.sliderValue[0]) &&
                (this.dataArray[i].sampleDate.getTime() <= this.sliderValue[1])) {
                tempArray[[m]].push(this.dataArray[i])
                timeArray.push(this.dataArray[i])
              }
            }
          }
        }
        d3.selectAll('path.line').remove()
        multydataFilteredArray = tempArray
        if (tempArray.length > 0) {
          this.multyUpdate()
        } else {
          window.alert('Non ci sono dati sufficienti per questa visualizzazione')
        }
      },

      measure () {
        tempArray = [[], [], [], [], [], [], [], [], [], []]
        scaleArray = []
        timeArray = []
        for (let m = 0; m < this.locationArray.length; m++) {
          this.location = this.locationArray[m]
          for (let i = 0; i < this.dataArray.length; i++) {
            if ((this.dataArray[i].measure === this.measure) && (typeof this.dataArray[i].value === 'number')) {
              this.unit=this.dataArray[i].unit
              scaleArray.push(this.dataArray[i])
              if ((this.dataArray[i].location === this.location) &&
                (this.dataArray[i].sampleDate.getTime() >= this.sliderValue[0]) &&
                (this.dataArray[i].sampleDate.getTime() <= this.sliderValue[1])) {
                tempArray[[m]].push(this.dataArray[i])
                timeArray.push(this.dataArray[i])
              }
            }
          }
        }
        d3.selectAll('svg > *').remove()
        console.log(this.unit)

        g = svg.append('g')
          .attr('transform', 'translate(' + margin.left +
            ', ' + margin.top + ')')

        xLabel = g.append('text')
          .attr('class', 'x axisLabel')
          .attr('y', height + 50)
          .attr('x', width / 2)
          .attr('font-size', '20px')
          .attr('text-anchor', 'middle')
          .text('Time')
        yLabel = g.append('text')
          .attr('class', 'y axisLabel')
          .attr('transform', 'rotate(-90)')
          .attr('y', -60)
          .attr('x', -170)
          .attr('font-size', '20px')
          .attr('text-anchor', 'middle')
          .text(this.measure)


        xAxisCall = d3.axisBottom().scale(x)
        // Append asse x
        xAxis = g.append('g')
          .attr('class', 'x axis')
          .attr('transform', 'translate(0,' + height + ')')

        // define the yaxis
        yAxisCall = d3.axisLeft().scale(y)
        // Append asse y
        yAxis = g.append('g')
          .attr('class', 'y axis')

        multydataFilteredArray = tempArray
        this.multyUpdate()
      },

      sliderValue () {
        tempArray = [[], [], [], [], [], [], [], [], [], []]
        scaleArray = []
        timeArray = []
        for (let m = 0; m < this.locationArray.length; m++) {
          this.location = this.locationArray[m]
          for (let i = 0; i < this.dataArray.length; i++) {
            if ((this.dataArray[i].measure === this.measure) && (this.dataArray[i].location === this.location)) {
              scaleArray.push(this.dataArray[i])
              if ((this.dataArray[i].location === this.location) &&
                (this.dataArray[i].sampleDate.getTime() >= this.sliderValue[0]) &&
                (this.dataArray[i].sampleDate.getTime() <= this.sliderValue[1])) {
                tempArray[[m]].push(this.dataArray[i])
                timeArray.push(this.dataArray[i])
              }
            }
          }
        }

        multydataFilteredArray = tempArray

        x = d3.scaleTime()
          .range([0, width])
          .domain(d3.extent(timeArray, function (d) {
            return d.sampleDate
          }))

        y = d3.scaleLinear()
          .range([height, 0])
          .domain([d3.min(scaleArray, function (d) {
            return d.value
          }),
            d3.max(scaleArray, function (d) {
              return d.value
            })])

        // domini delle scale
        xAxisCall.scale(x)
        xAxis.transition(this.transition()).call(xAxisCall)
        yAxisCall.scale(y)
        yAxis.transition(this.transition()).call(yAxisCall)

        for (let i = 0; i < multydataFilteredArray[[i]].length; i++) {
          // eslint-disable-next-line eqeqeq
          if (multydataFilteredArray[[i]].length != 0) {
            let dataArray = multydataFilteredArray[i]
            let id = dataArray[0].location

            line = d3.line()
              .x(function (d) {
                return x(d.sampleDate)
              })
              .y(function (d) {
                return y(d.value)
              })
            // domini delle scale

            line = d3.line()
              .x(function (d) { return x(d.sampleDate) })
              .y(function (d) { return y(d.value) })
            d3.select('#' + id)
              .transition(this.transition())
              .attr('d', line(dataArray))
          }
        }
      }
    }
  }

</script>

<style scoped>

  #checkboxLocation{
    float: left;
    margin-right: 1%;
    mask-border-repeat: repeat;
  }

  #chart-area svg {
    margin-left: auto;
    margin-right: auto;
    display: block;
  }

  .isActiveBoonsri{
    color: purple;
  }
  .isActiveKannika{
    color: red;
  }
  .isActiveChai{
    color: blue;
  }
  .isActiveKohsoom{
    color: yellow;
  }
  .isActiveSomchair{
    color: #089514;
  }
  .isActiveSakda{ color: #43cbd5;}
  .isActiveBusarakhan{ color: #c8c86a;}
  .isActiveTansanee{ color: #8d6695;}
  .isActiveAchara{ color: #525c95;}
  .isActiveDecha{ color: #95911c;}


  .axis {
    font: 10px sans-serif;
  }

  .axis path,

  .axis line {
    fill: none;
    stroke: #D4D8DA;
    stroke-width: 2px;
    shape-rendering: crispEdges;
  }

  #selections {
    padding-top: 10px;
    padding-bottom: 10px;
  }

  #sliderWrapper{
    text-align: center;
  }

  #selectionWrapper{
    text-align: center;
  }

  #dataSelection{

  }

  #container{
    display: inline-block;
    padding: 5px 5px;
    border: 2px solid #aed6f1;
    border-radius: 15px;
    margin: auto;
  }

  #sliderContainer{
    display: inline-block;
    padding: 5px 5px;
    border: 2px solid #aed6f1;
    border-radius: 15px;
    box-sizing: border-box;
    padding: 12px;
    max-width: 1028px;
    margin: auto;
  }

  #checkboxContainer{
    display: inline-block;
    padding: 5px 5px;
    border: 2px solid #aed6f1;
    border-radius: 15px;
    max-higth: 50%;
  }

  #measureContainer{
    display: inline-block;
    padding: 5px 5px;
    border: 2px solid #aed6f1;
    border-radius: 15px;
  }

  .active{
    color: red;
  }

  #labelOne{
    margin-right: 350px;
  }

  .line {
    fill: none;
    stroke-width: 3px;
    stroke: grey;
  }
</style>
