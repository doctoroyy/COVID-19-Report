<template>
  <div id="VaccineCountries">
    <van-swipe
      :loop='false'
      @change="chartsChange"
      class="my-swipe"
      ref="vaccine_swipe2"
      :touchable="true"
      indicator-color="transparent"
    >
      <van-swipe-item>
        <div class="container_charts countries_trend"></div>
      </van-swipe-item>
      <van-swipe-item>
        <div class="container_charts countries_trend2"></div>
      </van-swipe-item>
    </van-swipe>
    <swipeButton
      ref="vaccine_swipe_btn2"
      :currentIndex="currentIndex"
      @toggleSwipe="toggleSwipe"
      class="swipeButton"
      :btnList="btnList"
    ></swipeButton>
  </div>
</template>

<script>
import {
  getVaccineCountries,
} from "network/vaccineInformation";
import swipeButton from "components/private/swipeButton"; // 图表切换组件

export default {
  name: 'VaccineCountries',
  data () {
    return {
      getVaccineCountries: {
        // 世界各国疫苗数据
        totalTrend: {
          // 总趋势
          country: [],
          trend: [],
        },
        perHundredTrend: {
          // 百人比率
          country: [],
          trend: [],
        },
      },
      currentIndex: 0,
      btnList: [
        '累计接种情况',
        '每百人接种情况'
      ]
    }
  },
  created () {
    getVaccineCountries().then((res) => {
      // 发起网络请求
      let result = res.data.data.VaccineTrendData;
      for (const str in result.totalTrend) {
        this.getVaccineCountries.totalTrend.country.push(str);
        this.getVaccineCountries.totalTrend.trend.push(
          this.selectData(result.totalTrend[str])
        );
      }
      for (const str in result.perHundredTrend) {
        this.getVaccineCountries.perHundredTrend.country.push(str);
        this.getVaccineCountries.perHundredTrend.trend.push(
          this.selectData(
            result.perHundredTrend[str],
            "total_vaccinations_per_hundred"
          )
        );
      }
    });
  },
  components: {
    swipeButton,
  },
  methods: {
    selectData(dataArray, objName = "total_vaccinations") {  // 筛选合适的数据
      for (let i = dataArray.length - 1; i > -1; i--) {
        if (dataArray[i][objName] != -1) {
          return dataArray[i][objName];
        }
      }
    },

    setLineChartsOption(country, data, text, unit) {  // 设置图表选项
      let that = this
      return {
        color: ['#1d99d3'],
        title: {
          text,
          textStyle: {
            fontSize: 14,
          },
          top: '-1%',
          left: "1%",
        },
        tooltip: {
          trigger: "axis",
          axisPointer: {
            type: "shadow",
          },
          textStyle: {
            color: "#FFF", // 文字的颜色
            fontSize: '11',    // 文字字体大小
          },
          padding: [7],
          backgroundColor: "rgba(0, 0, 0, 0.623)",
          borderColor: "transparent",
          formatter: function (params) {
            let str = `国家：${params[0].name}</br>${params[0].marker} ${that.setUnitChar(params[0].value)}${unit}`
            return str;
          },
        },
        grid: {
          top: "7%",
          left: "4%",
          right: "6%",
          bottom: "3%",
          containLabel: true,
        },
        xAxis: {
          type: "value",
          boundaryGap: [0, 0.01],
          axisLabel: {
            textStyle: {
              fontSize:11,
            },
            formatter: function(val) {
              return that.setUnitChar(val)
            }
          },
        },
        yAxis: {
          type: "category",
          data: country,
          axisLabel: {
            textStyle: {
              fontSize: 11,
            },
          },
        },
        series: [
          {
            name: "",
            type: "bar",
            data,
          },
        ],
      };
    },

    chartsChange(index) {  // 切换图表
      this.$refs.vaccine_swipe_btn2.setIndex(index);
      this.loadChart(index)
    },

    toggleSwipe(index) {  // 切换图表
      this.$refs.vaccine_swipe2.swipeTo(index); // 切换轮播图
      this.loadChart(index)
    },

    loadChart(index) {  // 加载图表
      switch (index) {
        case 0:
          let myChart1 = this.$echarts.getInstanceByDom(document.querySelector(".countries_trend"))
          
          if (myChart1 == null) {
            myChart1 = this.$echarts.init(document.querySelector(".countries_trend"), null, {renderer: 'svg'})
          }
          console.log(1);
          myChart1.setOption(this.setLineChartsOption(this.getVaccineCountries.totalTrend.country, this.getVaccineCountries.totalTrend.trend, '各国疫苗累计接种','剂'))
          break;
        case 1:
          let myChart2 = this.$echarts.getInstanceByDom(document.querySelector(".countries_trend2"))
          if (myChart2 == null) {
            myChart2 = this.$echarts.init(document.querySelector(".countries_trend2"), null, {renderer: 'svg'})
          }
          myChart2.setOption(this.setLineChartsOption(this.getVaccineCountries.perHundredTrend.country, this.getVaccineCountries.perHundredTrend.trend, '各国每百人接种情况','剂'))
          break;
        default:
          break;
      }
    }
  },
  mounted () {
    
  }
}

</script>
<style scoped>
#VaccineCountries{
  margin-bottom: var(--marginB);
}
.container_charts{
  width: 100%;
  height: 280px;
}
</style>