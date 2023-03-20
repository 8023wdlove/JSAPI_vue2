<template>
    <div>
      <div id="container">
        <div class="region_list">
          <div class="region-list-bg">
            <div
              v-for="(item, i) in mapListSetting"
              :key="i"
              :class="[
                'region-list_block',
                'boxCardBlock',
                item.selected ? 'activeListBlock' : '',
              ]"
              @click="selectDelivery(i)"
            >
              <div v-if="mapListSetting.length != 1">
                <a class="close-modal-sub closeSmall" v-on:click.stop="RemoveItem(i)">
                  ×
                </a>
              </div>
              <div class="displayRow alignItemsCenter jscontentbet mb10">
                <label
                  class="block-label"
                  :style="{ borderColor: item.colorStyle }"
                >
                  <span
                    class="block-label_span"
                    :style="{ background: item.colorStyle, opacity: 0.2 }"
                  ></span>
                </label>
                <el-input
                  v-model="item.name"
                  clearable
                  placeholder="请输入内容"
                  style="width: 150px"
                />
              </div>
              <div class="displayRow alignItemsCenter jscontentbet mb10">
                <label class="region-list_block-label">起送价：</label>
                <el-input
                  v-model="item.minPriceout"
                  placeholder="00.00"
                  style="width: 110px"
                  @blur="regExpChange(item.minPriceout,'minPriceout',i)"
                  oninput="value=value.replace(/[^0-9.]/g,'').replace(/^(\-)*(\d+)\.(\d\d).*$/,'$1$2.$3')"
                >
                  <template slot="prepend">￥</template>
                </el-input>
              </div>
              <div class="displayRow alignItemsCenter jscontentbet" v-show="compute_type!='2'">
                <label class="region-list_block-label">配送费：</label>
                <el-input
                  v-model="item.pscost"
                  placeholder="00.00"
                  style="width: 110px"
                  @blur="regExpChange(item.pscost,'pscost',i)"
                  oninput="value=value.replace(/[^0-9.]/g,'').replace(/^(\-)*(\d+)\.(\d\d).*$/,'$1$2.$3')"
                >
                  <template slot="prepend">￥</template>
                </el-input>
              </div>
              <div class="region-list_block-arrow">
              </div>
            </div>
            <div class="addPsqy" v-if="mapListSetting.length <= 10">
              <button
                class="region-list_add"
                type="button"
                @click="addOpenPoly()"
              >
                添加配送区域
              </button>
            </div>
          </div>
        </div>
        <div class="region_type">
          <div class="region_type_contain">
            <div class="region-type_row displayRow alignItemsCenter">
              <label class="region-type_label">配送区域：</label>
              <div class="mr20">
                <el-radio v-model="drawtype" label="0" @change="selectedTypeRadio">
                  按半径框选
                </el-radio>
                <el-radio v-model="drawtype" label="1" @change="selectedTypeRadio">
                  自定义框选
                </el-radio>
              </div>
              <div class="ml20">
                <label class="region-type_label">半径距离：</label>
                <div class="region-type_custom" v-if="drawtype=='1'">
                  区域内
                </div>
                <span v-if="drawtype=='0'">
                  <el-input  v-model="circleArea"  style="width: 70px"  @change="changeCircleArea"/>
                  公里内
                </span>
              </div>
            </div>
          </div>
        </div>
  
        <div class="fullscreen" @click="clickFun">
          <!-- <vab-icon :icon="isFullscreen ? 'fullscreen-exit-fill' : 'fullscreen-fill'" style="font-size:20px"/> -->
        </div>
        
      </div>
    </div>
  </template>
  <script>
  import AMapLoader from '@amap/amap-jsapi-loader'
  // import screenfull from 'screenfull'
  
  export default {
    name: 'MapEdit',
    props: {
      //引用组件页面传来的值
      areas: {
        type: Array,
        default: function () {
          return []
        },
      },
      //中心点
      center: {
        type: Array,
        default: function () {
          return [116.403322, 39.920255]
        },
      },
      //绘制类型
      compute_type: {
        type: String,
        default: function () {
          return "2"
        },
      }
    },
    data() {
      return {
        isFullscreen:false,
        polyEditor: null,  //当前编辑的矢量图
        polygonPaths: [],  //编辑后的多边形对象
        map: null,   //初始化地图
        polygons: [],  //绘制的矢量图数组
        mapListIndex:0, //当前选择的区域index
        mapListSetting: [
          {
            name: '', //区域名称
            minPriceout: '0.00', //起送价
            pscost: '0.00',  //配送费
            selected: true,
            locations_object: [],
            colorStyle: '#1791fc',
            area_type:'1', //配送范围类型 0按半径 1自定义范围
            inkm:'1.000'  //半径
          },
        ],
        initia_locations_object:[], //根据中心点生成的初始位置
        //区域颜色color
        mapColor: [
          { color: '#1791fc' },
          { color: 'rgb(45, 166, 65)' },
          { color: 'rgb(133, 199, 0)' },
          { color: 'rgb(237, 106, 12)' },
          { color: 'rgb(250, 171, 12)' },
          { color: 'rgb(212, 42, 141)' },
          { color: 'rgb(138, 63, 212)' },
          { color: 'rgb(250, 210, 12)' },
          { color: 'rgb(17, 160, 173)' },
          { color: 'rgb(212, 0, 0)' },
        ],
        drawtype: '1', //1 自定义框选， 0 按半径框选
        circleArea:'1.000', 
      }
    },
    watch: {
      'areas':function(){
        //console.log(this.areas);  
      }
    },
    created(){
      console.log(this.areas);
      if(!this.areas) return;
      if(this.areas.length>0){
        this.areas.forEach((element, i) => {
          element.area_type=element.area_type.toString()
          if(i==0){
            this.mapListIndex=i
            element.selected = true
          }else{
            element.selected = false
          }
          if(i<this.areas.length){
            element.colorStyle = this.mapColor[i].color
          }
        })
        this.mapListSetting=this.areas
      }
    },
    async mounted() {
      var that=this
      await this.init() //初始化Map
  
      setTimeout(function(){
        //created事件里，若areas值长度大于0，则使用areas传来的数据，反之执行默认初始化数据;
        that.mapListSetting.forEach((element, i) => {
          if(i<that.mapListSetting.length){
            that.drawtype = element.area_type
            that.circleArea =  element.inkm
            that.initAreas(window.AMap, i)
            
            if(i+1==that.mapListSetting.length){
              that.drawtype = that.mapListSetting[0].area_type
            }
          }
          if (i==0) {
            that.drawtype = element.area_type
            that.initPoly(window.AMap,i)
          } 
          
        })
       
      },2000)
      
    },
    
    methods: {
      //点标记
      addMarker(AMap) {
        var startMarker = new AMap.Marker({
          position: new AMap.LngLat(this.center[0],this.center[1]),
        });
        this.map.add(startMarker);
      },
      // 简单实现绘制正方形，side 正方形边长，单位米
      // 若需要绘制其他矩形，需自行修改代码
      GetFourPoint(AMap,latLng,side){
        var lng=latLng[0]
        let lat=latLng[1]
        path();
        function path(){
          const centerPoint = new AMap.LngLat(lng, lat);
          const upLeftPoint = centerPoint.offset(-side / 2, side / 2);
          const upRightPoint = centerPoint.offset(side / 2, side / 2);
          const leftBottomPoint = centerPoint.offset(-side / 2, -side / 2);
          const rightBottomPoint = centerPoint.offset(side / 2, -side / 2);
          return [
            [upRightPoint.lng, upRightPoint.lat],
            [upLeftPoint.lng, upLeftPoint.lat],
            [leftBottomPoint.lng, leftBottomPoint.lat],
            [rightBottomPoint.lng, rightBottomPoint.lat],
          ];
        }
        this.initia_locations_object=path()  //根据中心点生成的初始位置
        if(!this.areas || this.areas.length==0){
          this.mapListSetting[0].locations_object=path()
        }
        //回显的数据locations_object为null时，默认赋值初始值
        this.mapListSetting.forEach(function(e){
          if(!e.locations_object){
            e.locations_object=path()
          }
        })
        // console.log(path());
      },
      async init() {
        await this.initMap()
      },
      async initMap() {
        let AMap = await AMapLoader.load({
          key: '8a8c4fd6c5d8cfa6abcb17dc345d9a5d',
          version: '2.0',
          plugins: [
            'AMap.PolygonEditor',
            'AMap.Autocomplete',
            'AMap.PlaceSearch',
            'AMap.Scale',
            'AMap.OverView',
            'AMap.ToolBar',
            'AMap.MapType',
            'AMap.PolyEditor',
            'AMap.CircleEditor',
            'AMap.Geolocation',
            'AMap.Geocoder',
            'AMap.AMapUI',
            'AMap.ControlBar',
            'AMap.Marker'
          ],
        })
        window.AMap = AMap
        console.log(AMap, 'AMapAMapAMap')
        this.map = new AMap.Map('container', {
          center: this.center,
          resizeEnable: true,
          zoom: 14,
          plugin: [
            //一些工具插件
            {
              pName: 'MapType', //地图类型
              defaultType: 0,
              events: {
                init(instance) {console.log(instance)},
              },
            },
          ],
        }) 
        
       
        this.map.addControl(new AMap.ControlBar({position:{ top: "10px", left: 90, right: "10px", } }));
        this.map.addControl(new AMap.ToolBar({position:{ top: '110px', left: 0, right: '40px', } }));
        //this.map.addControl(new AMap.Scale( {position:{ right: '10px', bottom:"10px"} } )); 比例尺
        // 缩放地图到合适的视野级别
        this.map.setFitView()
        
        this.GetFourPoint(AMap,this.center, 2000) 
        this.addMarker(AMap)
        this.map.on('click', this.showInfoClick);
      },
      //地图点击事件用来测试）
      showInfoClick(e){
        var text = '您在 [ '+e.lnglat.getLng()+','+e.lnglat.getLat()+' ] 的位置单击了地图！'
        console.log(text)
      },
      initAreas(AMap,i,type) {
        if(type=="editType"){
          this.polyEditor.close()
          this.map.remove([this.polygons[i]])
          this.polygons.splice(i, 1)
        }
        
        let col = this.mapListSetting[i].colorStyle
        var polygon = null
        if (this.drawtype == '1') {
          let path = this.mapListSetting[i].locations_object
          if (path.length <= 0) {
            return
          }
  
          polygon = new AMap.Polygon({
            path: path,
            strokeColor: col,
            strokeWeight: 2,
            strokeOpacity: 1,
            fillOpacity: 0.1,
            fillColor: col,
            zIndex: 50,
            bubble: true,
          })
        } else {
          //console.log(i,col)
          polygon = new AMap.Circle({
            center: this.center,
            radius: this.circleArea*1000, //半径
            borderWeight: 3,
            strokeColor: col,
            strokeOpacity: 1,
            strokeWeight: 2,
            // strokeOpacity: 1,
            fillOpacity: 0.4,
            strokeDasharray: [10, 10],
            fillColor: col,
            zIndex: 50,
          })
        }
        
        if(type=="editType"){
          this.polygons.splice(i, 0, polygon)
        }else{
          this.polygons.push(polygon)
        }
        
        if (this.polygons.length <= 0) {
          return
        }
        //地图上新增矢量多边形
        // console.log(this.polygons)
        this.map.add(this.polygons)
      },
      initPoly(AMap, i) {
        var that = this
        let opts = {
          editOptions:{strokeColor: this.mapListSetting[i].colorStyle,  fillColor: this.mapListSetting[i].colorStyle,} 
        }
        console.log(this.polygons, 'this.polygon')
        if (this.polygons.length > 0) {
          if(this.drawtype=="1"){
            this.polyEditor = new AMap.PolygonEditor( this.map,  this.polygons[i], opts)
          }else{
            this.polyEditor =  new AMap.CircleEditor(this.map, this.polygons[i], opts)
          }
        } else {
          if(this.drawtype=="1"){
            this.polyEditor = new AMap.PolygonEditor(this.map)
          }else{
            this.polyEditor =  new AMap.CircleEditor(this.map)
          }
        }
        that.polyEditor.open()
        this.delivery_area()
  
       //自适应地图缩放
        that.map.setFitView(that.polygons[i], false, [130, 60, 100, 60])
        that.polyEditor.on('adjust',function(){
          that.map.setFitView(that.polygons[i], false, [130, 60, 100, 60])
        })
  
        //关闭多边形编辑polygonEditor.close()触发该方法；
        if(this.drawtype=="1"){
          this.polyEditor.on('adjust', function (event) {
            // event.target 即为编辑后的多边形对象,event.target.getPath()得到编辑完成后的点数组
            let pointArr = event.target.getPath()
            this.polygonPaths = []
            for (let i = 0; i < pointArr.length; i++) {
              this.polygonPaths.push([
                pointArr[i].lng,
                pointArr[i].lat,
              ])
            }
            that.mapListSetting[i].locations_object=this.polygonPaths
            that.delivery_area()
            console.log('polygonPaths', this.polygonPaths)
          })
        }else{
          this.polyEditor.on('adjust', function (event) {
            //console.log(event);
            that.circleArea = (event.radius)/1000
            that.mapListSetting.forEach((element, index) => {
              if (that.mapListIndex == index) {
                element.inkm = that.circleArea
                that.delivery_area()
              }
            })
          })
        }
      },
      //添加配送区域
      addOpenPoly() {
        if (this.mapListSetting.length >= 10) return
  
        this.mapListSetting.forEach((element) => {
          this.mapColor.forEach((e,j)=>{
            if(element.colorStyle==e.color){
              this.mapColor.splice(j,1)
            }
          })
        })
  
        var that = this
        that.polyEditor.close()
        this.mapListSetting.push({
          name: '',
          minPriceout: '0.00',
          pscost: '0.00',
          selected: false,
          locations_object: this.initia_locations_object,
          colorStyle : that.mapColor[0].color
        })
        this.circleArea = '1.000'
        this.initAreas(window.AMap,this.mapListSetting.length - 1)
        this.mapListSetting.forEach((element, i) => {
          if (i + 1 == this.mapListSetting.length) {
            element.selected = true
            element.area_type = that.drawtype
            element.inkm = that.circleArea
            that.initPoly(window.AMap, i)
            that.mapListIndex=i
          } else {
            element.selected = false
          }
        })
      },
      //选择编辑的区域
      selectDelivery(i) {
        var that = this
        //console.log(i)
        that.mapListIndex=i,
        that.polyEditor.close()
        that.mapListSetting.forEach((element, index) => {
          if (i == index) {
            element.selected = true
            that.drawtype = element.area_type
            that.circleArea = element.inkm
            that.initPoly(window.AMap, i)
          } else {
            element.selected = false
          }
        })
        // console.log(that.mapListSetting[i].selected);
      },
      //删除编辑区域
      RemoveItem(i) {
        this.mapListSetting.splice(i, 1)
        this.map.remove([this.polygons[i]])
        this.polygons.splice(i, 1)
        
        this.mapColor.push({color:this.mapListSetting[i].colorStyle})
        
        //console.log(i, i+1,this.mapListSetting.length)
        if(i==this.mapListSetting.length){
          this.mapListSetting[i-1].selected=true
          this.mapListIndex=i-1
        }
        if(this.polygons.length==1) {
          this.selectDelivery(0)
        }
      },
      //选择绘制类型
      selectedTypeRadio() {
        this.mapListSetting.forEach((element, index) => {
          if (this.mapListIndex == index) {
            element.area_type=this.drawtype
          }
        })
        //console.log(this.mapListIndex);
        this.initAreas(window.AMap, this.mapListIndex, "editType")
        this.initPoly(window.AMap,this.mapListIndex)
      },
      //更改半径距离
      changeCircleArea(){
        this.mapListSetting.forEach((element, index) => {
          if (this.mapListIndex == index) {
            element.inkm=this.circleArea
            this.initAreas(window.AMap,this.mapListIndex, "editType")
            this.initPoly(window.AMap,this.mapListIndex)
          }
        })
      },
      //保存（这里是向父组件传值）
      delivery_area(){
        this.$emit('getPolygonMap', this.mapListSetting)
      },
      //全屏
      clickFun(){
        // const element = document.getElementById('container');//指定全屏区域元素
        //       if (!screenfull.isEnabled) {
        //     this.$baseMessage('开启全屏失败', 'error', 'vab-hey-message-error')
        // }
        
        // this.isFullscreen=!screenfull.isFullscreen
        // if (screenfull.isEnabled) {
        //     screenfull.request(element);
        //  }
        // screenfull.toggle()
      },
      //保留两位小数
      regExpChange(val,str,i){
        this.$set(this.mapListSetting[i], str, Number(val).toFixed(2))
      }
    },
  }
  </script>
  
  <style>
  .region-list_block .el-input-group__prepend {
    padding: 0 10px !important;
  }
  </style>
  <style>
  .displayRow{
    display:flex;
    flex-flow:row;
  }
  .displayColumn{
    display:flex;
    flex-flow:column; 
  }
  .alignItemsCenter{
    align-items: center;
  }
  .jscontentbet{
    justify-content: space-between;
  }
  #container {
    width: 100%;
    height: 600px;
    position: relative;
    min-width: 660px;
    background: #f3f3f3;
    color: #333333;
  }
  .region_list {
    width: 210px;
    max-height: 96%;
    position: absolute;
    background: #fff;
    top: 12px;
    left: 12px;
    z-index: 2;
    overflow-y: scroll;
    box-sizing: border-box;
    box-shadow: 0 0 5px #afafaf;
  }
  .region_list::-webkit-scrollbar {
    width: 0px;
    opacity: 0;
  }
  .region-list_block {
    position: relative;
    width: 100%;
    box-sizing: border-box;
    padding: 12px 22px 12px 12px;
    border: 1px solid transparent;
    border-bottom-color: #ebedf0;
  }
  .addPsqy {
    width: 100%;
  }
  .region-list_add {
    display: block;
    padding: 0;
    margin: 10px auto;
    font-size: 14px;
    width: 196px;
    color: #323233;
    line-height: 32px;
    background: #fff;
    outline: none;
    border: 1px solid #dcdee0;
    border-radius: 2px;
    cursor: pointer;
  }
  .activeListBlock {
    border-color: #1890ff;
  }
  .block-label {
    width: 15px;
    height: 15px;
    border: solid 1px #ccc;
  }
  .block-label_span {
    width: 100%;
    height: 100%;
    display: block;
  }
  .closeSmall {
    top: 4px;
    right: 4px;
    width: 15px;
    height: 15px;
    font-size: 12px;
    line-height: 16px;
  }
  /** */
  .region_type {
    width: 100%;
    position: absolute;
    z-index: 1;
    padding: 12px 12px 12px 244px;
    box-sizing: border-box;
  }
  .region_type_contain {
    padding: 10px 12px 10px 20px;
    display: flex;
    flex-direction: column;
    background: hsla(0, 0%, 100%, 0.85);
    box-shadow: 0 0 5px #afafaf;
    font-size: 14px;
    max-width: 565px;
  }
  .region-type_row {
    padding: 10px 0;
    white-space: nowrap;
    overflow-x: auto;
  }
  .region-type_custom{
    display: inline-block;
  }
  .region-type_custom::before{
    display: inline-block;
    content: "";
    width: 22px;
    height: 22px;
    margin-top: -4px;
    margin-right: 8px;
    vertical-align: middle;
    background-size: 100%!important;
    background-position: 50%;
    background-repeat: no-repeat;
    /* background: url('../../../../public/img/icons/area.png'); */
  }
  // 选中角标
  .activeListBlock .region-list_block-arrow{
    position: absolute;
    right: 0;
    bottom: 0;
    border: 10px solid #1890ff;
    border-top-color: transparent;
    border-left-color: transparent;
  }
  .region-list_block-arrow::before,.region-list_block-arrow::after{
    position: absolute;
    display: block;
    content: "";
    height: 2px;
    background: #fff;
  }
  .activeListBlock .region-list_block-arrow::before {
    width: 5px;
    left: -3px;
    bottom: -8px;
    transform: rotate(35deg);
  }
  .activeListBlock .region-list_block-arrow::after {
    width: 12px;
    left: -1px;
    bottom: -5px;
    transform: rotate(130deg);
  }
  .fullscreen{
    position: absolute;
    right: 20px;
    bottom: 20px;
    z-index: 1;
    border-radius: 50%;
    display: block;
    background: #fff;
    width: 40px;
    height: 40px;
    line-height: 40px;
    text-align: center;
  }
  </style>
  
  