<template>
  <div>
    <div class="box" id="main"></div>
  </div>
</template>

<script>
import echarts from "echarts";
import { mapGetters, mapActions } from "vuex";
export default {
  computed: {
    ...mapGetters({
      list: "cate/list"
    })
  },
  methods: {
    ...mapActions({
      reqList: "cate/reqList"
    })
  },
  watch: {
    list: {
      handler() {
        if (this.list.length > 0) {
          
      
          let myChart = echarts.init(document.getElementById("main"));

         
          var option = {
            title: {
              text: "项目数据"
            },
            tooltip: {},
            legend: {
              data: ["子分类数量"]
            },
            xAxis: {
              data: this.list.map(item=>item.catename),
            },
            yAxis: {},
            // series 数据
            series: [
              {
                name: "子分类数量",
                type: "bar",
                data:this.list.map(item=>item.children?item.children.length:0)
              },
            ]
          };

          myChart.setOption(option);
        }
      },
      deep: true
    }
  },
  mounted() {
    
    this.reqList();
  }
};
</script>

<style scoped>
.box {
  width: 80%;
  margin: 40px auto;
  height: 800px;
}
</style>

