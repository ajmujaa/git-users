<template>
  <section class="container">
    <form>
      <label for="username">Users of Github:</label>
      <input id="username" class="form-control" autocomplete="off" type="text" placeholder="Type to search...">
      <typeahead v-model="model" :limit="limit" target="#username" :async-function="queryFunction" item-key="git_url">
        <template slot="item" slot-scope="props">
          <li v-for="(item, index) in props.items" :key="index" :class="{active:props.activeIndex===index}">
            <a role="button" @click="props.select(item), updateChart()">
              <span v-html="item.name"></span>
            </a>
          </li>
        </template>
      </typeahead>
    </form>
    <alert class="alert" v-show="model">You selected {{model.contributors_url}}</alert>
    <div class="chart">
      <canvas id="contributors-data"></canvas>
    </div>
  </section>
</template>

<script>
import axios from "axios";
import Chart from "chart.js";
import contributorChart from "@/chart-data.js";

export default {
    data () {
      return {
        model: '',
        limit: 100,
        contributorChart: contributorChart
      }
    },
    mounted() {
      this.createChart('contributors-data', this.contributorChart);
    },
    methods: {
      queryFunction (query, done) {
        axios.get('https://api.github.com/search/users?q=' + query)
          .then(res => {
            if(res.data.items !== '') {
              const repos_url = res.data.items[0].repos_url;
              axios.get(repos_url).then(response => {
                  done(response.data);
              })
              .catch(err => {
                alert("Something went wrong. Please search again");
              })
            }
          })
          .catch(err => {
            alert("Something went wrong. Please search again");
            // any error handler
          })
      },
      createChart(chartId, chartData) {
        const ctx = document.getElementById(chartId);
        const contChart = new Chart(ctx, {
          type: chartData.type,
          data: chartData.data,
          options: chartData.options,
        });
      },
      updateChart() {
        axios.get(this.model.contributors_url)
          .then(response => {
            const data = response['data'];
            if(data) {
              this.removeChartData();
              data.forEach(d => {
                this.contributorChart.data.labels.push(d.login);
                this.contributorChart.data.datasets.forEach(dataset => {
                  dataset.data.push(d.contributions)
                });
              });
              this.createChart('contributors-data', this.contributorChart);     
            }
        })
        .catch(err => {
          console.log("Something went wrong. Please try again");
        });
      },
      removeChartData() {
        this.contributorChart.data.labels.pop();
        this.contributorChart.data.datasets.forEach(dataset => {
          dataset.data.pop()
        });
        this.createChart('contributors-data', this.contributorChart);     
      }
    }
  }
  
</script>

<style lang="scss" scoped>
.container {
  margin-top: 50px;
  .alert{
    margin: 20px 0;
  }
  .chart {
    height:20vh; 
    width:60vw;
    margin: 0 auto;
  }
}
</style>
