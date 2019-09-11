<template>
    <div class="wrapper">
        <div id="line" style="width:100%; height:400px; margin-top:20px"></div>
        <el-row>
            <div style="margin-top:40px; margin-left:10%">
                <div id="bar" style="float:left; width:45%; height:400px;"></div>
                <div id="pie" style="float:left; margin-left:10%; width:35%; height:400px;"></div>
            </div>
        </el-row>
        <el-row>
            <div style="display:block; margin-top:40px; margin-left:10%">
                <div id="calendar" style="float:left; width:90%; height:400px"></div>
            </div>
        </el-row>
        <el-row>
            <div style="display:block; margin-top:40px; margin-left:10%; margin-bottom:40px">
                <HomeTabel title="Top10 接口排行" :data="topList" :columns="columns" />
            </div>
        </el-row>
        <br />
    </div>
</template>
<script>
import utils from '@/tool/utils'
import api from '@/common/api'
import axios from 'axios'
import echarts from 'echarts/lib/echarts'
import 'echarts/lib/chart/line'
import 'echarts/lib/chart/bar'
import 'echarts/lib/chart/pie'
import 'echarts/lib/chart/heatmap'
import 'echarts/lib/component/legend'
import 'echarts/lib/component/title'
import 'echarts/lib/component/tooltip'
import 'echarts/lib/component/visualMap'
import HomeTabel from './HomeTable.vue'

export default {
    data() {
        return {
            topList: [],
            columns: [
                {
                    prop: 'name',
                    label: '接口名称'
                },
                {
                    prop: 'code',
                    label: '接口编码'
                },
                {
                    prop: 'url',
                    label: '接口路径'
                },
                {
                    prop: 'platform',
                    label: '所属平台'
                },
                {
                    prop: 'count',
                    label: '调用次数'
                }
            ]
        }
    },

    components: {
        HomeTabel: HomeTabel
    },

    mounted() {
        this.fetchEchart()
    },

    methods: {
        tableRowStyle({row, rowIndex}) {
            return 'background-color:#FFF'
        },

        tableHeaderColor({row, column, rowIndex, columnIndex}) {
            if (rowIndex === 0) {
                return 'background-color:#389ffd; color:#FFF; font-weight:500'
            }
        },

        handleLineGraph() {
            const lineGraph = echarts.init(document.getElementById('line'))

            api.get(
                `/req/bus/dashboard/v1/dashboard/month_count`
            ).then(response => {
                let xAxisData = [], seriesData = []

                for (const item of response.result) {
                    xAxisData.push(item.month)
                    seriesData.push(item.count)
                }

                const lineOption = {
                    title: {
                        text: '总调用次数',
                        x: 'center',
                        y: 'top',
                        textAlign: 'left',
                        textStyle: {
                            fontWeight: "bold",
                            color: "#333",
                            fontSize: 18
                        }
                    },
                    tooltip: {},
                    xAxis: {
                        type: 'category',
                        boundaryGap: false,
                        data: xAxisData
                    },
                    yAxis: {
                        type: 'value'
                    },
                    series: [{
                        data: seriesData,
                        type: 'line',
                        color: "#389ffd",
                        areaStyle: {},
                        itemStyle: {  
                            normal: {    
                                lineStyle: {  
                                    color: '#389ffd'
                                }    
                            }  
                        }
                    }]
                }

                lineGraph.setOption(lineOption)
            })
        },

        handleBarGraph() {
            const barGraph = echarts.init(document.getElementById('bar'))

            api.get(
                `/req/bus/dashboard/v1/dashboard/platform/count`
            ).then(response => {
                let xAxisData = [], seriesData = []

                for (const item of response.result) {
                    xAxisData.push(item.platform)
                    seriesData.push(item.count)
                }

                const barOption = {
                    title: {
                        text: '平台调用次数',
                        textStyle: {
                            fontWeight: "bold",
                            color: "#333",
                            fontSize: 18
                        }
                    },
                    tooltip: {},
                    xAxis: {
                        data: xAxisData
                    },
                    yAxis: {},
                    series: [{
                        name: '平台调用次数',
                        type: 'bar',
                        data: seriesData,
                        color: "#389ffd",
                        itemStyle: {  
                            normal: {
                                lineStyle: {  
                                    color: '#389ffd'
                                }
                            }  
                        }
                    }]
                }

                barGraph.setOption(barOption)
            })
        },

        handleWeight(result) {
            let weightSortAry = []
            let weightTotal = 0

            for (let k = 0; k < result.length; k++) {
                weightTotal += parseInt(result[k].count)
            }

            for (let n = 0; n < result.length; n++) {
                weightSortAry.push({
                    count: result[n].count,
                    platform: result[n].platform,
                    weight: (parseInt(result[n].count) / weightTotal).toFixed(2)
                })
            }

            weightSortAry.sort(function(a, b){
                return b.weight - a.weight
            })

            return weightSortAry
        },

        handlePieGraph() {
            const pieGraph = echarts.init(document.getElementById('pie'))
            const colorCard = ['#36cbcb','#f2637b', '#389ffd', '#fbd437']

            api.get(
                `/req/bus/dashboard/v1/dashboard/platform/count`
            ).then(response => {
                let seriesData = [], legendData = []
                const weightedAry = this.handleWeight(response.result)

                for (let i = 0; i < weightedAry.length; i++) {
                    const x = i % colorCard.length

                    legendData.push(weightedAry[i].platform)

                    seriesData.push({
                        value: weightedAry[i].count,
                        name: weightedAry[i].platform,
                        itemStyle: {
                            color: colorCard[x]
                        }
                    })
                }

                const pieOption = {
                    title: {
                        text: '应用分类',
                        textStyle: {
                            fontWeight: "bold",
                            color: "#333",
                            fontSize: 18
                        }
                    },
                    tooltip: {
                        trigger: 'item',
                        formatter: '{a} <br/>{b} : {c} ({d}%)'
                    },
                    legend: {
                        orient: 'vertical',
                        left: 'right',
                        data: legendData
                    },
                    series: [
                        {
                            name: '访问来源',
                            type: 'pie',
                            radius: '55%',
                            center: ['50%', '60%'],
                            data: seriesData,
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

                pieGraph.setOption(pieOption)
            })
        },

        handleTable() {
            api.get(
                `/req/bus/dashboard/v1/dashboard/action/top`
            ).then(response => {
                this.topList = response.result
            })
        },

        handleCalendarGraph() {
            api.get(
                `/req/bus/dashboard/v1/dashboard/action/top`
            ).then(response => {
                this.topList = response.result
            })
        },

        handleDay(day) {
            switch (day) {
                case '1':
                    return '星期一'
                    break
                case '2':
                    return '星期二'
                    break
                case '3':
                    return '星期三'
                    break
                case '4':
                    return '星期四'
                    break
                case '5':
                    return '星期五'
                    break
                case '6':
                    return '星期六'
                case '7':
                    return '星期日'
                    break
            }
        },

        handleCalendarGraph() {
            const calendarGraph = echarts.init(document.getElementById('calendar'))

            api.get(
                `/req/bus/dashboard/v1/dashboard/week/count`
            ).then(response => {

                const hours = [
                    '01', '02', '03', '04', '05', '06', '07', '08', '09', '10', '11', '12',
                    '13', '14', '15', '16', '17', '18', '19', '20', '21', '22', '23', '00'
                ]

                const dates = [
                    '星期一',
                    '星期二', 
                    '星期三',
                    '星期四',
                    '星期五',
                    '星期六',
                    '星期日'
                ]

                let data = []

                for (const item of response.result) {
                    for (const subItem of item.hourCount) {
                        data.push([
                            this.handleDay(item.day),
                            subItem.hour,
                            subItem.count
                        ])
                    }
                }

                data = data.map(function (item) {
                    return [item[1], item[0], item[2] || '-']
                })

                const calendarOption = {
                    title : {
                        text: '调用频次分布图',
                        textStyle: {
                            fontWeight: "bold",
                            color: "#333",
                            fontSize: 18
                        }
                    },
                    tooltip: {
                        position: 'top'
                    },
                    animation: false,
                    grid: {
                        height: '50%',
                        y: '20%'
                    },
                    xAxis: {
                        type: 'category',
                        data: hours,
                        splitArea: {
                            show: true
                        }
                    },
                    yAxis: {
                        type: 'category',
                        data: dates,
                        splitArea: {
                            show: true
                        }
                    },
                    visualMap: {
                        min: 0,
                        max: 250,
                        calculable: true,
                        orient: 'horizontal',
                        left: 'center',
                        bottom: '15%',
                        inRange: {
                            color: ['#8FC9FF', '#389ffd']
                        }
                    },
                    series: [{
                        name: 'Punch Card',
                        type: 'heatmap',
                        data: data,
                        label: {
                            normal: {
                                show: true
                            }
                        },
                        itemStyle: {
                            emphasis: {
                                shadowBlur: 10,
                                shadowColor: 'rgba(0, 0, 0, 0.5)'
                            }
                        }
                    }]
                }

                calendarGraph.setOption(calendarOption)
            })
        },

        fetchEchart() {
            this.handleLineGraph()
            this.handleBarGraph()
            this.handlePieGraph()
            this.handleTable()
            this.handleCalendarGraph()
        },

        getVirtulData(year) {
            year = year || '2017'

            let date = +echarts.number.parseDate(year + '-01-01')
            let end = +echarts.number.parseDate((+year + 1) + '-01-01')
            let dayTime = 3600 * 24 * 1000
            let data = []

            for (let time = date; time < end; time += dayTime) {
                data.push([
                    echarts.format.formatTime('yyyy-MM-dd', time),
                    Math.floor(Math.random() * 1000)
                ])
            }

            return data
        }
    }
}
</script>
<style lang="scss" scoped>
table {
    width: 80%;
    border: 1px solid #c6c6c6;
}
tr td {
    padding: 20px;
    border: 1px solid #c6c6c6;
}
</style>