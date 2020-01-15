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
      <div id="menu">
      <button b-tooltip.hover title="The history mode allows you to view in sequence all the data available for a specific measurement" class="storyButton" v-if="visiblePlay" v-on:click='visible=true; visiblePlay=false; visibleSlide=false; pausePause=true; storyModeButton = true; storyMode()' id="play">Play story mode</button>
        <button  class="storyButton" v-if="visible & pausePause"  v-on:click='pausePause=true; storyModeButton=false; visiblePlay=true; visibleSlide=true; ' id="pause">Pause story mode</button>
        <div id="focusData" v-if="showFocusData===true">
        <label >Focus on relevant data</label>
        <input type="checkbox" v-model="storymodeInfo" v-on:click="showLocation=!showLocation">
      </div>
        <!-- eslint-disable -->
      <select v-if="showLocation" v-model="location" id="locationDiv"  class="style-chooser" placeholder="Location" v-on:change="storyModeButton=false; selectorActive=true">
        <option v-for='locations in sinLocation'>{{locations}}</option>
      </select>
      <select v-if="showLocation" v-model="measure"  class="style-chooser"
              placeholder="Measure"
              :options="['Components', 'CSS / Variables', 'Slots']">
        <option v-for='measures in sinMeasure' onclick="storyModeButton=false">{{measures}}</option>
        <!-- eslint-enable -->
      </select>
        <button v-if="visible" class="storyButton" id="restart" v-on:click="storyModeButton=false; pausePause=true;  restart=true; visiblePlay=true; visibleSlide=false">Restart story mode</button>
        <button v-if="visible & visibleSlide & checkShowFocusData"  class="storyButton" id="AddSlide" v-on:click="storyModeButton=false; addSlide=true">Add one slide</button>
        <button v-if="visible & visibleSlide & checkShowFocusData" class="storyButton" id="subtractSlide" v-on:click="storyModeButton=false; subtractSlide=true">Subtract one slide</button>

      </div>
    </div>
    <div id="col-md-12">
      <div id="chart-area"></div>
      <div id="info">
        <!-- eslint-disable -->
        <div id="infoText" v-for='texts in textArray' v-if="texts.measure===measure && storymodeInfo===false"><h3>{{texts.measure}}</h3> {{texts.text}}</div>
        <div id="infoTextstory" v-for='texts in textArrayStory' v-if="texts.location===location && storymodeInfo===true" ><h3>{{texts.title}}</h3> {{texts.text}}</div>
        <div id="infoLink" v-if="showLocation">Usefull link: <br>
          <a v-for='texts in textArray' v-if="texts.measure===measure" href='texts.link'>{{texts.link}}</a></div>
      </div>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
  import Vue from 'vue'
  import * as d3 from 'd3'
  import VueSlider from 'vue-slider-component'
  import 'vue-slider-component/theme/default.css'
  Vue.component('VueSlider', VueSlider)

  // eslint-disable-next-line

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
  let alert = false
let locationReset= 'Boonsri'
let measureReset= 'Water temperature'

  let parseTime = d3.timeParse('%Y-%m-%d')
  let formatTime = d3.timeFormat('%d/%m/%Y')
  // var bisectDate = d3.bisector(function (d) { return d.date }).left

  export default {
    name: 'MC2LineChart',
    data () {
      return {
        visibleSlide: false,
        visiblePlay: true,
        showFocusData: true,
        storymodeInfo: false,
        visible: false,
        subtractSlide: false,
        addSlide: false,
        restart: false,
        slide: 0,
        linkDisplay: null,
        selectorActive: false,
        checkShowFocusData: true,
        pausePause: true,
        showLocation: true,
        textArrayStory: [
          {'title': 'Molecola x',
            'location': 'Boonsri',
          'text': 'Poca roba'},
          {'title': 'Tante senza dati',
            'location': 'Kannika',
            'text': 'Poca roba'},
          {'title': 'Chai',
            'location': 'Chai',
            'text': 'Picchetto e plateau'},
          {'title': 'Kohsoom',
            'location': 'Kohsoom',
            'text': 'Picco violento'},
          {'title': 'Somchair',
            'location': 'Somchair',
            'text': 'Picco violento e plateau'},
        ],
        textArray: [{
          'measure': 'Water temperature',
          'text': 'A temperature expresses hot and cold, as measured with a thermometer. In physics, hotness is a body \' s ability to impart energy as heat to another body that is colder.\n' +
            'In a body in which there are processes of chemical reaction and flow of matter, temperature may vary over its parts, and over time, as measured by a suitably small and rapidly responding thermometer, and may depend also on the match of the processes to the characteristics of the thermometer.\n' +
            'There are various temperature scales that nearly or approximately agree with one another, but differ slightly because of the various characteristics of particular thermometric substances. The most commonly used scales are the Celsius scale (formerly called centigrade) (denoted °C), Fahrenheit scale (denoted °F), and Kelvin scale (denoted K).',
          'link': 'https://en.wikipedia.org/wiki/Temperature'
        },
          {
            'measure': 'Dissolved oxygen',
            'text': 'Dissolved oxygen refers to the level of free, non-compound oxygen present in water or other liquids. It is an important parameter in assessing water quality because of its influence on the organisms living within a body of water. In limnology (the study of lakes), dissolved oxygen is an essential factor second only to water itself ¹.  A dissolved oxygen level that is too high or too low can harm aquatic life and affect water quality.\n' +
              'Non-compound oxygen, or free oxygen (O2), is oxygen that is not bonded to any other element. Dissolved oxygen is the presence of these free O2 molecules within water.The bonded oxygen molecule in water (H2O) is in a compound and does not count toward dissolved oxygen levels. One can imagine that free oxygen molecules dissolve in water much the way salt or sugar does when it is stirred.',
            'link': 'https://www.fondriest.com/environmental-measurements/parameters/water-quality/dissolved-oxygen/'
          },
          {
            'measure': 'emergencyText',
            'text': 'No data to show for this parameters. Change settings and keep exploring',
            'link': 'All work and no data makes Jack a dull boy...'
          }],
        storyModeButton: false,
        max: parseTime('2016-12-31').getTime(),
        min: parseTime('1998-01-11').getTime(),
        step: 86400000, // One day
        sliderValue: [parseTime('1998-01-11').getTime(), parseTime('2016-12-31').getTime()],
        formatter: v => `${formatTime((new Date(v)))}`,
        dataArray: this.readyArray,
        dataFilteredArray: [],
        sinLocation: this.singleLocation,
        storymodeLocation: ['Boonsri', 'Kannika', 'Chai', 'Kohsoom', 'Somchair'],
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
          if ((this.dataArray[i].measure === this.measure) && (this.dataArray[i].location === this.location)) {
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
        .text('Price (USD)')

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
        yAxis.transition(this.transition()).call(yAxisCall)

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
      },

      transition: function () {
        return d3.transition().duration(1000)
      },

      storyMode: function () {
        let array=[]
        this.showFocusData=false
        if (this.storymodeInfo === false) {
          // eslint-disable-next-line no-unused-expressions
           array = this.sinLocation.slice(this.slide)
          console.log(array.length)
        } else {
          this.checkShowFocusData=false
          this.measure= 'Methylosmoline'
          array = this.storymodeLocation
          console.log(array.length)
        }
          const it = array[Symbol.iterator]() // convenient for yeilding values
          const int = setInterval(() => { // time interval
            const next = it.next() // next value
            if ((!next.done) && (this.storyModeButton === true)) { // done = true when the end of array reached

              this.location = next.value // concatenate word to the string
              this.slide++
              console.log(this.slide)
            } else {
              if (this.restart === true) {
                console.log('restart')
                this.slide = 0
                this.location = 'Boonsri'
                this.measure = 'Water temperature'
                this.restart = false
                this.slide = 0
                this.visible = false
                this.showFocusData=true
                this.checkShowFocusData=true
                clearInterval(int)
              }
              if (this.selectorActive === true) {
                console.log('restart')
                this.slide = 0
                this.slide = 0
                this.selectorActive = false
                this.visible = false
                clearInterval(int)
              }
              if (this.addSlide === true) {
                this.slide++
                if (array[this.slide] === undefined) {
                  this.addSlide = false
                  this.location = 'Boonsri'
                  this.measure = 'Water temperature'
                  this.slide = 0
                  clearInterval(int)
                } else {
                  this.location = array[this.slide - 1]
                  this.addSlide = false
                }
              }
              if (this.subtractSlide === true) {
                this.slide--
                if ((array[this.slide] === undefined) || (this.slide<0)) {
                  this.subtractSlide = false
                  this.location = 'Boonsri'
                  this.measure = 'Water temperature'
                  this.slide = 0
                  clearInterval(int)
                } else {
                  this.location = array[this.slide - 1]
                  this.subtractSlide = false
                }
              }
            }
          }, 3000) // interval duration, 1s

        }
    },

    watch: {

      location () {
        svg.selectAll('image').remove()
        tempArray = []
        scaleArray = []
        for (let i = 0; i < this.dataArray.length; i++) {
          if ((this.dataArray[i].measure === this.measure) && (typeof this.dataArray[i].value === 'number')) {
            scaleArray.push(this.dataArray[i])
            if ((this.dataArray[i].measure === this.measure) && (this.dataArray[i].location === this.location) &&
              (this.dataArray[i].sampleDate.getTime() >= this.sliderValue[0]) &&
              (this.dataArray[i].sampleDate.getTime() <= this.sliderValue[1])) {
              tempArray.push(this.dataArray[i])
            }
          }
        }
        this.dataFilteredArray = tempArray
        if (tempArray.length > 0) {
          this.update()
        } else {
          this.measure = 'emergencyText'
        }
      },

      measure () {
        svg.selectAll('image').remove()
        tempArray = []
        scaleArray = []
        for (let i = 0; i < this.dataArray.length; i++) {
          if ((this.dataArray[i].measure === this.measure) && (typeof this.dataArray[i].value === 'number')) {
            scaleArray.push(this.dataArray[i])
            if ((this.dataArray[i].measure === this.measure) && (this.dataArray[i].location === this.location) &&
              (this.dataArray[i].sampleDate.getTime() >= this.sliderValue[0]) &&
              (this.dataArray[i].sampleDate.getTime() <= this.sliderValue[1])) {
              tempArray.push(this.dataArray[i])
            }
          }
        }
        this.dataFilteredArray = tempArray
        if (tempArray.length > 0) {
          this.update()
        } else {
          this.measure = 'emergencyText'

        }
      },

      sliderValue () {
        svg.selectAll('image').remove()
        tempArray = []
        scaleArray = []
        for (let i = 0; i < this.dataArray.length; i++) {
          if ((this.dataArray[i].measure === this.measure) && (typeof this.dataArray[i].value === 'number')) {
            scaleArray.push(this.dataArray[i])
            if ((this.dataArray[i].measure === this.measure) && (this.dataArray[i].location === this.location) &&
              (this.dataArray[i].sampleDate.getTime() >= this.sliderValue[0]) &&
              (this.dataArray[i].sampleDate.getTime() <= this.sliderValue[1])) {
              tempArray.push(this.dataArray[i])
            }
          }
        }
        this.dataFilteredArray = tempArray
        if (tempArray.length > 0) {
          this.update()
        } else {
          this.measure = 'emergencyText'
          // eslint-disable-next-line eqeqeq
        }
      }
    },
  }
</script>

<style scoped>

  #chart-area svg {
    margin-left: auto;
    margin-right: auto;
    display: block;
  }

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

  #selections .col-md-4 {
    padding-top: 10px;
    padding-bottom: 10px;
  }

  #sliderWrapper{
    text-align: center;
  }

  .storyButton{
    display: inline-block;
    padding: 5px 5px;
    border: 2px solid #aed6f1;
    border-radius: 15px;
    box-sizing: border-box;
    padding: 12px;
    max-width: 1028px;
    margin: auto;
    background: white;
  }

  #sliderContainer{
    display: inline-block;
    padding: 5px 5px;
    border: 2px solid #aed6f1;
    border-radius: 15px;
    box-sizing: border-box;
    padding: 12px;
  }

  #labelOne{
    margin-right: 350px;
  }

  #menu{
    display: inline-block;
    padding: 5px 5px;
    border: 2px solid #aed6f1;
    border-radius: 15px;
    box-sizing: border-box;
    padding: 12px;
    margin: 0.5%;
  }




  #focusData{
    display: inline-block;
    padding: 5px 5px;
    border: 2px solid #aed6f1;
    border-radius: 15px;
    box-sizing: border-box;
    padding: 12px;
    max-width: 1028px;
    margin: auto;
  }


  #col-md-12{
    display: inline-block;
    padding: 5px 5px;
    border-radius: 15px;
    box-sizing: border-box;
    padding: 12px;
    max-width: 1200px;
  }
  #chart-area{
    margin-right: 10px;
    float: left;
    border: 2px solid #aed6f1;
    border-radius: 15px;
    box-sizing: border-box;}
  #info{
    overflow: hidden;
    border-radius: 15px;
    box-sizing: border-box;
    position: relative}

  #infoTextstory,
  #infoText,
  #infoLink{
    top: 0;
    left: 0;
    border: 2px solid #aed6f1;
    border-radius: 15px;
    margin: 5px  }
  #infoLink {
    z-index: 10;
    border: 2px solid #aed6f1;
    border-radius: 15px;
    margin: 5px
  }

  .line {
    fill: none;
    stroke-width: 3px;
    stroke: grey;
  }

  .style-chooser {
    background: #aed6f1;
    border: none;
    color: #394066;
    text-transform: lowercase;
    font-variant: small-caps;
  }
  #container{
    display: inline-block;
    padding: 5px 5px;
    border: 2px solid #aed6f1;
    border-radius: 15px;
    margin: auto;
  }

</style>
