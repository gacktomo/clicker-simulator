<script>
import { Line, mixins } from 'vue-chartjs'
import * as ChartAnnotation from "chartjs-plugin-annotation";
const { reactiveProp } = mixins

export default {
  extends: Line,
  mixins: [reactiveProp],
  props: ['options'],
  mounted() {
    Chart.plugins.register(ChartAnnotation);
    this.addPlugin([ChartAnnotation]),
    this.renderChart(this.chartData, this.options)
  },
}
</script>
