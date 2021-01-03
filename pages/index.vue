<template>
  <div class="container">
    <div>
      <Chart
        v-if="loaded"
        :width="600"
        :chartData="chartData"
        :options="options"
      />
    </div>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import Chart from '../components/LineChart.vue'

export default Vue.extend({
  components: { Chart },
  data() {
    return {
      loaded: false,
      result: [],
      chartData: {
        labels: ['0'],
        datasets: [
          {
            label: '所持金',
            data: [0],
            backgroundColor: 'rgba(255, 200, 0, 1)',
            borderColor: 'rgba(255, 200, 0, 1)',
            fill: false,
            type: 'line',
            pointRadius: 0,
          },
          {
            label: '1秒あたりの獲得ポイント(タップ考慮)',
            data: [0],
            backgroundColor: 'rgba(176,224,230, 1)',
            borderColor: 'rgba(176,224,230, 1)',
            fill: false,
            type: 'line',
            pointRadius: 0,
          },
          {
            label: '１時間あたりの獲得ポイント(放置時)',
            data: [0],
            backgroundColor: 'rgba(135,206,250, 1)',
            borderColor: 'rgba(135,206,250, 1)',
            fill: false,
            type: 'line',
            pointRadius: 0,
          },
          {
            label: '感情レベル',
            data: [100],
            backgroundColor: 'rgba(240, 9, 22, 1)',
            borderColor: 'rgba(240, 9, 22, 1)',
            fill: false,
            type: 'line',
            pointRadius: 0,
            yAxisID: "y-axis-2",
          },
        ],
      },
      options: {
        responsive: true,
        scales: {
          yAxes: [
            {
              id: 'y-axis-1',
              type: 'logarithmic',
              position: 'left',
              scaleLabel: {
                display: true,
                labelString: 'ポイント',
                fontSize: 18
              },
            },
            {
              id: 'y-axis-2',
              type: 'linear',
              position: 'right',
              scaleLabel: {
                display: true,
                labelString: '感情',
                fontSize: 18
              },
              ticks: {
                min: 0,
              },
            },
          ],
        },
      },
    }
  },
  mounted() {
    const priceAdditionRate = 0.15
    const tapPerSec = 5
    const bonusInterval = 300
    const bonusTime = 15
    const bonusRate = 10
    let pointPerSec = 1
    let currentBalance = 0
    let emotion = 100;
    const emotionKeepTime = 60;
    const emotionDecrementPerSec = emotion / emotionKeepTime;

    const promoTable = [
      { price: 50, addPoint: 1, level: 1 },
      { price: 5000, addPoint: 5, level: 1 },
      { price: 50000, addPoint: 25, level: 1 },
      { price: 500000, addPoint: 125, level: 1 },
      { price: 5000000, addPoint: 625, level: 1 },
    ]
    const goodsTable = [
      { price: 100, isSold: false },
      { price: 5000, isSold: false },
      { price: 25000, isSold: false },
      { price: 50000, isSold: false },
      { price: 100000, isSold: false },
      { price: 200000, isSold: false },
    ]
    const getPrice = (price: number, level: number) => price + price * priceAdditionRate * (level - 1);
    const setEmotion = (point: number) => {
      emotion += emotionDecrementPerSec * 20;
      if(emotion > 100) emotion = 100;
    }

    for (let i = 1; i < 3000; i++) {
      // グッズ購入
      for(const item of goodsTable) {
        if(!item.isSold && item.price < currentBalance) {
          currentBalance -= item.price;
          item.isSold = true;
          setEmotion(emotionDecrementPerSec * 30);
        }
      }

      // 最も効率のよい強化が購入できれば実施
      const mostCostPerformancePromo = promoTable.reduce((a, b) => getPrice(a.price, a.level)/a.addPoint > getPrice(b.price, b.level)/b.addPoint ? b : a)
      if (mostCostPerformancePromo.price < currentBalance) {
        currentBalance -= mostCostPerformancePromo.price
        pointPerSec += mostCostPerformancePromo.addPoint
        mostCostPerformancePromo.level++;
        setEmotion(emotionDecrementPerSec * 10);
      }

      // ボーナス加点考慮
      let additionalPoint = pointPerSec * tapPerSec
      if ((i % bonusInterval < bonusTime && 20 < i) || (45 < i && i < 60)) {
        additionalPoint *= bonusRate
        setEmotion(emotionDecrementPerSec * 20);
      }
      currentBalance += additionalPoint

      // イベント頻度による感情グラフ
      if(emotion > 0) emotion -= emotionDecrementPerSec;

      this.chartData.labels.push(i + '')
      this.chartData.datasets[0].data.push(currentBalance)
      this.chartData.datasets[1].data.push(additionalPoint)
      this.chartData.datasets[2].data.push(pointPerSec * 60 * 60)
      this.chartData.datasets[3].data.push(emotion)
    }
    this.loaded = true
  },
})
</script>

<style>
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
