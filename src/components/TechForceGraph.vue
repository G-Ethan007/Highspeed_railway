<template>
  <div ref="chartRef" class="chart-container"></div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import * as echarts from 'echarts'
import relationshipData from '../../data/relationship.json'

const chartRef = ref(null)
let chart = null

// 数据转换函数：将层级数据转换为节点和边
const transformData = (data) => {
  const nodes = []
  const links = []
  
  const processNode = (node, parentId = null, inheritedTech = null) => {
    const nodeId = node.name
    const techSource = node.tech_source || inheritedTech
    nodes.push({
      id: nodeId,
      name: node.name,
      tech_source: techSource,
      type: node.type,
      symbolSize: node.children ? 20 : 10,
      itemStyle: {
        color: getTechColor(techSource)
      }
    })
    
    if (parentId) {
      links.push({
        source: parentId,
        target: nodeId,
        lineStyle: {
          color: '#aaa',
          width: 1
        }
      })
    }
    
    if (node.children) {
      node.children.forEach(child => processNode(child, nodeId, techSource))
    }
  }
  
  processNode(data)
  return { nodes, links }
}

// 获取技术来源对应的颜色
const getTechColor = (techSource) => {
    const techColors = {
        '庞巴迪系': '#4F94CD',
        '新干线系': '#6FCF97',
        '西门子系': '#A285D1',
        '阿尔斯通系': '#F2B84B',
        '自主创新系': '#EB5757'
    }
    return techColors[techSource] || '#999'
}

// 初始化图表
const initChart = () => {
  if (!chartRef.value) return
  
  chart = echarts.init(chartRef.value)
  const { nodes, links } = transformData(relationshipData)
  
  const option = {
    backgroundColor: '#fff',
    tooltip: {
      trigger: 'item',
      formatter: (params) => {
        const data = params.data
        let text = `<strong>${data.name}</strong>`
        if (data.tech_source) {
          text += `<br/>技术来源: ${data.tech_source}`
        }
        if (data.type) {
          text += `<br/>类型: ${data.type}`
        }
        return text
      }
    },
    series: [{
      type: 'graph',
      layout: 'force',
      data: nodes,
      links: links,
      roam: true,
      draggable: true,
      focusNodeAdjacency: true,
      force: {
        repulsion: 200,
        edgeLength: 100,
        gravity: 0.1
      },
      label: {
        show: true,
        position: 'right',
        fontSize: 12
      },
        emphasis: {
            scale: true,
            label: {
                fontWeight: 'bold',
                fontSize: 13
            },
            itemStyle: {
                borderWidth: 3,
                borderColor: '#000',
                shadowColor: 'rgba(0,0,0,0.3)',
                shadowBlur: 10
            }
        },
      lineStyle: {
        color: 'source',
        curveness: 0.3
      }
    }]
  }
  
  chart.setOption(option)
}

onMounted(() => {
  initChart()
  window.addEventListener('resize', () => chart?.resize())
})

onUnmounted(() => {
  chart?.dispose()
  window.removeEventListener('resize', () => chart?.resize())
})
</script>

<style scoped>
.chart-container {
  width: 100%;
  height: 600px;
}
</style>