<template>
<div class="mod-demo-echarts">
    <el-card>
        <div id="J_chartBarBox" class="chart-box"></div>
    </el-card>
    <p></p>
    <el-card>
        <div id="J_chartPieBox2" class="chart-box"></div>
    </el-card>
</div>
</template>

<script>
  import echarts from 'echarts'
  export default {
    data () {
      return {
        chartBar: null,
        chartPie: null,
        dataForm: {
          key: ''
        },
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false,
        gcname: [],
        gcnum: []
      }
    },
    mounted () {
      this.initChartBar()
      this.initChartPie()
    },
    activated () {
      // 由于给echart添加了resize事件, 在组件激活时需要重新resize绘画一次, 否则出现空白bug
      if (this.chartBar) {
        this.chartBar.resize()
      }
      if (this.chartPie) {
        this.chartPie.resize()
      }
    },
    methods: {
      // 柱状图
      initChartBar () {
        this.$nextTick(() => {
          this.$http({
            url: this.$http.adornUrl('/goods_category/goodscate/tblist'),
            method: 'get'
          }).then(({data}) => {
            var option = {
              title: {
                text: '粥铺种类数量图',
                left: 'center'
              },
              tooltip: {
                trigger: 'axis',
                axisPointer: {
                  type: 'shadow'
                }
              },
              grid: {
                bottom: 90
              },
              dataZoom: [{
                type: 'inside'
              }, {
                type: 'slider'
              }],
              xAxis: {
                data: data.gcname,
                silent: false,
                splitLine: {
                  show: false
                },
                splitArea: {
                  show: false
                },
                axisLabel: {
                  interval: 0,
                  rotate: 20, // 20度角倾斜显示(***这里是关键)
                  textStyle: {
                    color: '#000000'
                  }
                }
              },
              yAxis: {
                splitArea: {
                  show: false
                }
              },
              series: [{
                type: 'bar',
                data: data.gcnum,
                // Set `large` for large data amount
                large: true,
                // 柱图宽度
                barWidth: 40
              }]
            }
            this.chartBar = echarts.init(document.getElementById('J_chartBarBox'))
            this.chartBar.setOption(option)
            window.addEventListener('resize', () => {
              this.chartBar.resize()
            })
          })
        })
      },
       // 饼状图
      initChartPie () {
        this.$nextTick(() => {
          this.$http({
            url: this.$http.adornUrl('/goods_category/goodscate/tblist1'),
            method: 'get'
          }).then(({ data }) => {
            var option = {
              title: {
                text: '粥铺种类数量图',
                // subtext: '纯属虚构',
                x: 'center'
              },
              tooltip: {
                trigger: 'item',
                formatter: '{a} <br/>{b} : {c} ({d}%)'
              },
              legend: {
                orient: 'vertical',
                left: 'left',
                data: data.gcname
              },
              series: [
                {
                  name: '种类数',
                  type: 'pie',
                  radius: '55%',
                  center: ['50%', '60%'],
                  data: data.gcnum,
                  itemStyle: {
                    emphasis: {
                      shadowBlur: 10,
                      shadowOffsetX: 0,
                      shadowColor: 'rgba(0, 0, 0, 0.5)'
                    }
                  }
                }
              ]
            }
            console.log(data.gcname)
            this.chartBar = echarts.init(document.getElementById('J_chartPieBox2'))
            this.chartBar.setOption(option)
            window.addEventListener('resize', () => {
              this.chartBar.resize()
            })
          })
        })
      }
    }
  }
</script>

<style lang="scss">
  .mod-demo-echarts {
    > .el-alert {
      margin-bottom: 10px;
    }
    > .el-row {
      margin-top: -10px;
      margin-bottom: -10px;
      .el-col {
        padding-top: 10px;
        padding-bottom: 10px;
      }
    }
    .chart-box {
      min-height: 400px;
    }
  }
</style>
