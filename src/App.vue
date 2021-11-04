<template>
  <div class="body">
    <div id="container"></div>
    <div class="test">123</div>
  </div>
</template>

<script>
export default {
  mounted() {
    const that = this;
    // var position = new AMap.LngLat(116, 39); //标准写法
    const map = new AMap.Map("container", {
      zoom: 4,
      viewMode: "3D",
    });
    const district = new AMap.DistrictSearch({
      // 返回行政区边界坐标等具体信息
      extensions: "all",
      // 设置查询行政区级别为 区
      level: "district",
    });
    let polygons = [];
    const conf = [
      {
        area: ["北京市"],
        position: [116.310905, 39.992806], // 北京大学
      },
      // {
      //   area: ["河南省", "郑州市", "金水区"],
      //   position: [113.660197, 34.797381], // 金水区一地方
      // },
      // {
      //   area: ["河南省", "郑州市", "二七区"],
      //   position: [113.670838, 34.723206],
      // },
      // {
      //   adcode: 410307,
      //   area: ["河南省", "洛阳市", "偃师区"],
      //   position: [112.902364, 34.71017],
      // },
    ];
    let index = 0;
    // 查询区域，生成边界信息，调整合适的聚焦比例
    function step1(item) {
      const { area, adcode } = item;
      return new Promise((resolve) => {
        district.search(
          adcode || area[area.length - 1],
          function (status, result) {
            console.log(result);
            const bounds = result.districtList[0].boundaries;
            if (bounds) {
              for (let i = 0, l = bounds.length; i < l; i++) {
                polygons.push(
                  new AMap.Polygon({
                    map: map,
                    strokeWeight: 1,
                    path: bounds[i],
                    fillOpacity: 0.7,
                    fillColor: "#CCF3FF",
                    strokeColor: "#CC66CC",
                  })
                );
              }
              // 地图自适应
              map.setFitView(polygons);
              // 两秒后加Marker
              setTimeout(() => {
                resolve(item);
              }, 2000);
            }
          }
        );
      });
    }
    // 添加marker，放大至marker视角，清除轮廓
    function step2(item) {
      const marker = new AMap.Marker({
        position: item.position, //位置
      });
      map.add(marker);
      return new Promise((resolve) => {
        setTimeout(() => {
          map.setFitView([marker]);
          map.setZoom(18);
          // 清除轮廓
          for (var i = 0, l = polygons.length; i < l; i++) {
            polygons[i].setMap(null);
          }
          polygons = [];

          const infoWindow = new AMap.InfoWindow({
            isCustom: true,
            content: that.$el.querySelector(".test"),
            // offset: new AMap.Pixel(16, -45),
          });
          infoWindow.open(map, item.position);
          // 五秒后跳转到下一个点（轮廓）
          setTimeout(() => {
            resolve(infoWindow);
          }, 5000);
        }, 2000);
      });
    }
    // eslint-disable-next-line
    function keyframe(item) {
      step1(item)
        .then((item) => {
          return step2(item);
        })
        .then((infoWindow) => {
          index++;
          // next
          if (conf[index]) {
            infoWindow.close();
            keyframe(conf[index]);
          }
        });
    }
    keyframe(conf[index]);

    // 北京大学 lng :  116.310905, lat :  39.992806  北京市海淀区
    // 第0步：看上一个坐标是否存在，存在的话
    // 第一步：通过坐标，拿到所在区域（县/乡 > 市 > 省）  （本质上，在加点的时候必须带上 省市区 这样就好做了）1和2就合并了
    // 第二步：区域拿到 便捷轮廓，绘制轮廓
    // 第三步：（看是否清楚轮廓）

    // var infoWindow = new AMap.InfoWindow({
    //   //创建信息窗体
    //   // isCustom: true, //使用自定义窗体
    //   content:
    //     '<div style="color: red;width: 200px;height: 200px;border:1px solid red">信息窗体</div>', //信息窗体的内容可以是任意html片段
    //   offset: new AMap.Pixel(16, -45),
    // });
    // var onMarkerClick = function (e) {
    //   infoWindow.open(map, e.target.getPosition()); //打开信息窗体
    //   //e.target就是被点击的Marker
    // };
    // marker.on("click", onMarkerClick);
  },
};
</script>

<style lang="less">
html,
body,
.body {
  margin: 0;
  padding: 0;
  height: 100%;
}
#container {
  height: 100%;
  outline: 1px solid red;
}
.test {
  color: red;
  font-weight: bold;
}
</style>
