<template>
  <div id="container" style="width: 100%; height: 300px"></div>
</template>

<script setup>
import { getBlogContributeCount } from "@/api/web/index";
import * as echarts from "echarts";
import { onMounted, ref } from "vue";

const contributeDate = ref([]);
const blogContributeCount = ref([]);

async function initDate() {
  getBlogContributeCount().then((response) => {
    if (response.code == 200) {
      contributeDate.value = response.data.contributeDate;
      blogContributeCount.value = response.data.blogContributeCount;

      const container = document.getElementById("container");
      if (!container) {
        console.error("容器元素未找到");
        return;
      }

      let chart = echarts.init(container);
      console.log("ECharts实例化成功:", chart);

      let option = {
        title: {
          top: 30,
          text: "笔记贡献度",
          subtext: "一年内笔记提交的数量",
          left: "center",
          textStyle: {
            color: "#000",
          },
        },
        tooltip: {
          trigger: "item",
          formatter: function (params) {
            return params.data[0] + "<br>笔记数：" + params.data[1];
          },
        },
        legend: {
          top: "30",
          left: "100",
          data: ["笔记数", "Top 12"],
          textStyle: {
            color: "#000",
          },
        },
        calendar: [
          {
            top: 100,
            left: "center",
            range: contributeDate.value,
            splitLine: {
              show: true,
              lineStyle: {
                color: "#D3D3D3",
                width: 4,
                type: "solid",
              },
            },
            yearLabel: { show: false },
            dayLabel: {
              nameMap: ["周日", "周一", "周二", "周三", "周四", "周五", "周六"],
              textStyle: {
                color: "#000",
              },
              firstDay: 1,
            },
            monthLabel: {
              nameMap: "cn",
              textStyle: {
                color: "#000",
              },
            },
            itemStyle: {
              normal: {
                color: "#ffffff",
                borderWidth: 1,
                borderColor: "#D3D3D3",
              },
            },
          },
        ],
        series: [
          {
            name: "笔记数",
            type: "scatter",
            coordinateSystem: "calendar",
            data: blogContributeCount.value,
            symbolSize: function (val) {
              if (val[1] == 0) {
                return val[1];
              } else {
                let size = 8 + val[1] * 2;
                if (size > 18) {
                  size = 18;
                }
                return size;
              }
            },
            itemStyle: {
              normal: {
                color: "#2ec7c9",
              },
            },
          },
        ],
      };
      chart.setOption(option);
    }
  });
}

onMounted(() => {
  initDate();
});
</script>
