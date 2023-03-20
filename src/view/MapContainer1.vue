<template>
    <div class="home_div">
        <div class="map_title">
            <h3>JSAPI Vue2地图组件示例</h3>
        </div>
        <div id="container"></div>
    </div>
</template>
<script>
import AMapLoader from '@amap/amap-jsapi-loader';

export default {
    name: "Mapview",
    data() {
        return {
            //map:null,
        }
    },
    created() {

    },
    mounted() {
        this.initAMap();
    },
    methods: {

        initAMap() {
            AMapLoader.load({
                key: '8a8c4fd6c5d8cfa6abcb17dc345d9a5d',  //设置您的key
                version: "2.0",
                plugins: ['AMap.ToolBar', 'AMap.Driving'],
                AMapUI: {
                    version: "1.1",
                    plugins: [],

                },
                Loca: {
                    version: "2.0"
                },
            }).then((AMap) => {
                this.map = new AMap.Map("container", {
                    viewMode: "3D",
                    zoom: 20,
                    zooms: [2, 22],
                    center: [117.39, 39.9],
                });
                this.$nextTick(()=> {
                    this.renderLine(AMap)
                })
            }).catch(e => {
                console.log(e);
            })
        },
        renderLine(AMap) {
            const { map } = this
            // 创建一条折线覆盖物
            var path = [
                new AMap.LngLat("116.368904","39.913423"),
                new AMap.LngLat("116.382122","39.901176"),
                new AMap.LngLat("116.387271","39.912501"),
                new AMap.LngLat("116.398258","39.904600")
            ];
            var polyline = new AMap.Polyline({
                path: path,  
                borderWeight: 300, // 线条宽度，默认为 1
                strokeColor: 'blue', // 线条颜色
                lineJoin: 'round' // 折线拐点连接处样式
            });
            map.add(polyline);

            // 创建两个点标记
            var marker1 = new AMap.Marker({
                position: new AMap.LngLat(117.39, 39.9),   // 经纬度对象，如 new AMap.LngLat(116.39, 39.9); 也可以是经纬度构成的一维数组[116.39, 39.9]
                title: '北京'
            });
            var marker2 = new AMap.Marker({
                position: new AMap.LngLat(117.49, 39.9),   // 经纬度对象，如 new AMap.LngLat(116.39, 39.9); 也可以是经纬度构成的一维数组[116.39, 39.9]
                title: '北京1'
            });
            map.add(marker1);
            map.add(marker2);

            // 自动适配到合适视野范围
            // 无参数，默认包括所有覆盖物的情况
            map.setFitView();
            // 传入覆盖物数组，仅包括polyline和marker1的情况
            map.setFitView([polyline,marker1]);

        }
    }

}
</script>
<style  scoped>
.home_div {
    padding: 0px;
    margin: 0px;
    width: 100%;
    height: 100%;
    position: relative;
}

#container {
    padding: 0px;
    margin: 0px;
    width: 100%;
    height: 100%;
    position: absolute;
}

.map_title {
    position: absolute;
    z-index: 1;
    width: 100%;
    height: 50px;
    background-color: rgba(27, 25, 27, 0.884);

}

h3 {
    position: absolute;
    left: 10px;
    z-index: 2;
    color: white;
}
</style>