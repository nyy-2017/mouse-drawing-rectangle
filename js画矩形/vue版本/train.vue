<template>
    <div unselectable="on" onselectstart="return false;" style="-moz-user-select:none;">
        <h1>地铁线路实时动态图</h1>
        <div id="trainSelected">
            <div class="train">
                <div class="lines" v-for="(lines,index) in all" :value="lines.line.lineId" :key="index">
                    <div class="lines_title">{{ lines.line.lineName}}:</div>
                    <div class="Train_contains">
                        <div class="stations" 
                            :ref="station.tccStationId + index" 
                            v-for="(station,index) in lines.stationInfo" :key="index"  
                            :value="station.tccStationId" 
                            @mousemove="on_mousemove(station.tccStationId)"
                            :style="{width:100/(lines.stationInfo.length) + '%'}">
                            <span class="station_item">{{ station.stationSname}}</span>
                            <div class="bottom" v-if="index !== lines.stationInfo.length">
                                <div class="bottom1"></div>
                                <div class="circle" 
                                    @click="get_stationId(station.tccStationId,$event,index)" 
                                    :class="{'actived': active.indexOf( station.tccStationId) > -1}"></div>
                                <!-- <div class="circle" @click="get_stationId(station.tccStationId,$event),isshow=!isshow" :style="{background: active.indexOf( station.tccStationId) > -1 ? '#3CB371': '#1E90FF'}"></div> -->
                                <!-- <div class="center"></div> -->
                            </div>
                        </div>
                    </div>
                    <!-- 小车的位置 --> 
                    <div class="positions">
                        <div id="act_point"
                            v-for="(item,index) in positions" :key="index"
                            v-if="lines.line.lineId === item.lineId"
                            :style="{left: item.left + 'px', bottom: item.bottom + 'px'}"
                            :class="{'bg1':item.flag == true,'bg2':item.flag == false }">
                            <!-- 标题文字 -->
                            <Tooltip placement="top-end">
                                <div style="width:60px; height:22px;"></div>
                                <div slot="content">
                                    <p>{{'线路Id：' + item.showLineName}}</p>
                                    <p>{{'列车编号：' + item.trainName }}</p>
                                    <p>{{'车站Id：' + item.showStationName}}</p>
                                    <p>{{'到达时间：' + item.time}}</p>
                                </div>    
                            </Tooltip>
                        </div>
                    </div>
                </div> 
            </div>
        </div>
        <!--选中的车站 el-dialog  -->
        <el-dialog :visible.sync="modal1"  width="600px" title="选中的车站" :before-close="handleClose"> 
            <div>
                <br/>
                <div class="select">您所选中的车站是 <span>{{ this.start_name }}</span> 至 <span>{{ this.end_name }}</span></div>
                <br/>
            </div>
            <div slot="footer" style="padding: 5px 10px;">
                <el-Row type="flex" :gutter="24" justify="end">
                    <el-Col :span="8"><el-button type="primary" @click="ttsPlay">TTS直播</el-button></el-Col>
                    <el-Col :span="8" :push="1"><el-button type="primary" @click="recordPlay=false">预录制直播</el-button></el-Col>
                    <el-Col :span="8" :push="1"><el-button type="primary" @click="prescoPlay= false">实时预录制直播</el-button></el-Col>
                    <el-Col :span="8" :push="1"><el-button type="primary" @click="addArea = false">话筒直播</el-button> </el-Col>
                </el-Row>
            </div>
        </el-dialog>
    </div>
</template>
<script>
import allDate from './lineAndStation.json'
export default {
    components: {allDate},
    data(){
        return {
            new_Positions:[], // 
            all:null,
            choosedData:[],
            lines:[],
            stations:[],
            positions: [],
            active: [],
            //     
            new_arr: [],
            arr2:[],
            pressed: false, 
            start:'',
            start_name:"", // 开始站
            end:'',
            end_name:"", // 结束站
            pressData: null,
            modal1: false, //选中车站的modal 初始状态
            socket: []
        }
    },
    methods:{
        handleClose(done) {
             done();
            // this.$confirm('确认关闭？')
            // .then(_ => {
            //     done();
            // })
            // .catch(_ => {
            //     this.start_name = null;
            //     this.end_name = null;
            //     this.pressData = []; //圈中的值，初始化为空
            //     // this.on_mousemove = null; 
            // });
        },
        //tts直播
        ttsPlay(){    
        },
        //预录制直播
        recordPlay(){            
        },
        //实时预录制直播
        prescoPlay(){   
        },
        //话筒直播
        addArea(){  
        },
        //矩形滑动的高度
        scrollTop(params){
            var scrollTop=0;
            if(document.documentElement && document.documentElement.scrollTop){
                scrollTop=document.documentElement.scrollTop;
            }else if(document.body){
                scrollTop= document.body.scrollTop;
            }
            return scrollTop
        },
        //鼠标事件
        boxSelection(){
            this.pressData = []; //圈中的值，初始化
            var trainSelected = document.getElementById('trainSelected');
            // console.log("trainSelected:",trainSelected);
            // 鼠标按下时
            trainSelected.onmousedown = (e) =>{
                // console.log('开始',e , e.clientY, e.clientY);
                if(e.button ==2) return false; //禁止鼠标右键事件
                var posx = e.clientX;
                var posy = e.clientY;
                this.pressed = true; // 允许拖动
                var div = document.createElement("div");
                div.className = "tempDiv";
                div.style.left = e.clientX+"px";
                div.style.top = e.clientY + this.scrollTop() + "px";
                
                let preBoxs = document.getElementsByClassName('tempDiv')
                console.log('preBox', preBoxs)
                for(let i= 0; i < preBoxs.length; i++){
                    if(preBoxs[i]){
                        trainSelected.removeChild(preBoxs[i])
                    }
                }
                trainSelected.appendChild(div);
                // 鼠标指针移出时
                trainSelected.onmousemove = (ev) =>{
                      div.style.border= '3px dashed yellowgreen'
                    // div.style.left = Math.min(ev.clientX, posx) + "px";
                    // div.style.top = Math.min(ev.clientY, posy) + this.scrollTop() + "px";
                    div.style.left = Math.min(ev.clientX, posx) - 230 + "px";
                    div.style.top = Math.min(ev.clientY, posy) - 200 + this.scrollTop() + "px";
                    div.style.width = Math.abs(posx - ev.clientX)+"px";
                    div.style.height = Math.abs(posy - ev.clientY)+"px";
                    // 松开鼠标按钮时
                    trainSelected.onmouseup = (event) =>{
                        console.log(2)
                        // console.log('结束', event.clientX, event.clientY)
                        this.pressed = false; // 禁止拖动
                        if(div && div.parentNode){
                            div.parentNode.removeChild(div); // 移除框选的虚线框
                        }
                        
                        // trainSelected.onmousedown = null;
                        trainSelected.onmousemove = null;
                        trainSelected.onmouseup = null;

                        // 判断车站stationAid 和 stationBid 大小，决定谁先谁后
                        let AID = this.pressData[0]; //第1个
                        let BID = this.pressData[this.pressData.length-1]; //最后1个

                        let num_AID = Number( AID.slice(2));
                        let num_BID = Number( BID.slice(2));

                        // if(num_AID > num_BID){ //反方向，不会执行
                        // debugger;
                        //     this.start = BID;
                        //     this.end = AID;
                        // } 
                        if( num_BID > num_AID){ //正向
                            this.start = AID;
                            this.end = BID;
                        }
                        // console.log("end",this.pressData, this.pressData[0], this.pressData[this.pressData.length-1], Math.abs(posx - ev.clientX), Math.abs(posy - ev.clientY));
                        // 1.拿socket 返回的数据的 车站id 和 画的矩形的车站 id 判断，
                        // 2.如果在画的矩形区域范围内，就把trainid 取出来
                        // console.log("查找trainid:",this.positions);

                            this.new_Positions = []; // 清空缓存
                            this.positions.map(item=>{
                                let num_stationAId = Number( (item.stationAId).slice(2));
                                let num_stationBId = Number( (item.stationBId).slice(2));
                                //正向
                                if( num_stationBId > num_stationAId ){
                                    if( num_stationBId < num_BID && num_AID < num_stationAId ){
                                        //  socket返回的 列车id
                                        this.new_Positions.push(item.trainName);
                                    }
                                }else if( num_stationBId < num_stationAId){ //反向
                                    if( num_stationAId < num_BID && num_AID < num_stationBId ){
                                        //  socket返回的 列车id
                                        this.new_Positions.push(item.trainName);
                                    }
                                }
                            })
                        
                            this.new_Positions = [...new Set(this.new_Positions)];
                            // console.log("that.resultArr:",this.new_Positions);

                        //根据id 查找name
                        this.all.map(v=>{
                            if( !v.stationInfo) return false;
                            v.stationInfo.map(station=>{
                                if( this.start == station.tccStationId ){
                                    this.start_name = station.stationSname
                                }
                                if( this.end == station.tccStationId ){
                                    this.end_name = station.stationSname
                                }
                                // console.log("查找后的name:",this.start_name,this.end_name);
                            })
                        })
                        //如果选中的有值，弹层打开 
                        if( this.start_name && this.end_name && Math.abs(posx - ev.clientX) >90 &&  150 < Math.abs(posy - ev.clientY) && Math.abs(posy - ev.clientY)< 280){
                            this.modal1= true; // 打开弹层
                        }
                    }
                }
                //点击后---松开鼠标按钮时
                trainSelected.onmouseup = (event) =>{
                    this.pressed = false; // 禁止拖动
                    trainSelected.onmousemove = null;
                    trainSelected.onmouseup = null;
                    // div.parentNode.removeChild(div); // 移除框选的虚线框
                    // console.log(event,event.button);
                    if(event.button == 0 && event.button == 1 && event.button == 2){
                        div.parentNode.removeChild(div); // 移除框选的虚线框 
                    }
                    // console.log(1);
                    for(let i= 0; i < preBoxs.length; i++){
                        if(preBoxs[i]){
                            trainSelected.removeChild(preBoxs[i])
                        }
                    }
                } 
            }
        },
        //取出来：矩形里开始位置的Aid 和 结束位置的Bid
        on_mousemove(data){ // 鼠标移到这块区域，取stationid，存到数组this.pressData里
            // console.log("圈中的data:",data);
            if (!this.pressed) return false;
            this.pressData.push(data);
        },
        //点击获取当前车的位置的Id
        get_stationId(id,e){
            // let _index = this.choosedData.indexOf(id)
            // if(_index>-1) {
            //     e.target.style.backgroundColor = '#1E90FF';
            //     this.choosedData.splice(_index,1)
            // } else {
            //     e.target.style.backgroundColor = 'red';
            //     this.choosedData.push(id)
            // }
            // e.target.style.cursor = 'pointer'; 
            // console.log("id",id,this.choosedData,e.target);
 
        },
        //计算小车的具体位置信息
        setPositions() {
            this.active = [];
            this.all.map(line => {
                // console.log("line",line);
                let _stations = [];
                if ( line.stationInfo) {
                    line.stationInfo.map(station => {
                        _stations.push(station.tccStationId);
                    })
                    // console.log("_stations:",_stations);
                }
                // 计算出 left 的初始值
                // 判断socket接口里面返回的数组
                this.positions.map( v => { 
                    // console.log("vvvvv",v);
                    if (v.lineId === line.line.lineId) {
                        v.showLineName = line.line.lineName; // 新赋值的变量，页面显示用
                        // console.log("v.showLineName",v.showLineName);
                        let A_index = _stations.indexOf(v.stationAId)
                        let B_index = _stations.indexOf(v.stationBId)
                        if (A_index < B_index && B_index <_stations.length + 1) {
                            // 正向
                            v.flag = true
                        } else  {
                            // 反向
                            v.flag = false
                        }
                        // 计算A站台到始发站的距离
                        let div = this.$refs[v.stationAId + A_index];
                        if (div && div[0]) {
                            // console.log("0 of underfind:",this.$refs[v.stationBId + B_index]);
                            // console.log("div", this.$refs[v.stationAId+A_index][0],this.$refs[v.stationBId + B_index][0].offsetLeft); // html
                            
                            // 返回车站信息数据错误，没有此车站!!! 弹出 this.$alert
                            // if( this.$refs[v.stationBId + B_index] == undefined){
                            //     this.$alert("返回车站信息数据错误，没有此车站!!!","温馨提示：");
                            // }

                            let A_offsetLeft = parseInt(this.$refs[v.stationAId + A_index][0].offsetLeft);
                            let B_offsetLeft = parseInt(this.$refs[v.stationBId + B_index][0].offsetLeft);

                            v.left = A_offsetLeft + parseInt(v.locationFlag == 0 ? 0 : Math.abs(B_offsetLeft - A_offsetLeft) / 2);
                            v.bottom = v.flag ? 15 : -35; //小车在 线上正反方向的位置
                            // console.log("小车id：", v.trainId);
                            // console.log('this.$refs',A_index, B_index, v.lineId, v.stationAId, v.stationBId, A_offsetLeft, B_offsetLeft, v.left,v.bottom);
                        }
                        if(v.locationFlag == 0 ) {
                            this.active.push(v.stationAId);
                        }
                        // 鼠标放在--小车上显示的车站name
                        line.stationInfo.map(station => {
                            if(v.stationAId == station.tccStationId ){
                                v.showStationName = station.stationSname
                            }
                        })
                        // console.log("v.showStationName:",v.showStationName);     
                    }
                })
            })
            // console.log('this.active', this.active );
        },

        //调后台接口，取出多个（ip）和 （端口），然后同时连接
        // getMore_ipAndPort(){
        //     this.$axios({
        //         method: 'get',
        //         url: this.baseURL + '/train/TrainInfo/getTrainLocationInfo',
        //     }).then((res)=> {
        //         // console.log("ip -and -port",res.data);
        //         if(res.data.length >0){
        //             for(var i = 0; i <res.data.length; i++) {
        //                 this.connect(i,res.data[i]); //创建连接
        //             }
        //         }else{
        //             return false;
        //         }
        //     }).catch((error) =>{}) 
        // },
        // 多个ip 和 port 同时连接
        // connect(i,wsobj){
        //     if ("WebSocket" in window) {
        //         var host = "ws://"+wsobj.ip+":"+wsobj.port;
        //         this.socket[i] = new WebSocket(host);
        //         try {
        //             //连接事件
        //             let that = this
        //             this.socket[i].onopen = function(open) {
        //                 // console.log(wsobj+":连接已建立！");
        //                 that.socket[i].send('0x01000103')
        //             };
        //             //错误事件
        //             this.socket[i].onerror = function(error) {
        //                 this.$Message.error("socket错误：" + error.data);
        //             };
        //             // 放处理后的结果
        //             //消息事件
        //             this.socket[i].onmessage = function(message) {
        //                 //this.$Message.info(wsobj+"消息接收:"+msg.data);

        //                 // console.log( '消息接收:返回', message.data);
        //                 let returnArr1 = JSON.parse(message.data);

        //                 if(returnArr1.length <=0){
        //                     return false;
        //                 }else if(returnArr1.length >0 && returnArr1.length <=1 ){
        //                     //socket 没回只返回一趟列车的情况
        //                     returnArr1.map(v=>{
        //                         v.left = -8, v.bottom = -8, v.flag = true, v.showLineName = null, v.showStationName = null
        //                         if(that.new_arr.length==0){
        //                             let new_list = {};
        //                             new_list.trainId = returnArr1[0].trainId
        //                             new_list.trainName = returnArr1[0].trainName
        //                             new_list.lineId = returnArr1[0].lineId
        //                             new_list.showLineName = returnArr1[0].showLineName  // 新加的变量
        //                             new_list.left = returnArr1[0].left 
        //                             new_list.bottom = returnArr1[0].bottom
        //                             new_list.time = returnArr1[0].time 
        //                             new_list.stationAId = returnArr1[0].stationAId 
        //                             new_list.showStationName = returnArr1[0].showStationName  //  新加的变量
        //                             new_list.stationBId = returnArr1[0].stationBId
        //                             new_list.locationFlag = returnArr1[0].locationFlag 
        //                             that.new_arr.push(new_list);
        //                             that.arr2.push(returnArr1[0].trainId);
        //                         } else {
        //                             if (that.arr2.indexOf(v.trainId)>-1) {
        //                                 var index= that.arr2.indexOf(v.trainId)
        //                                 that.new_arr[index].lineId = v.lineId;
        //                                 that.new_arr[index].trainName = v.trainName;
        //                                 that.new_arr[index].showLineName = v.showLineName // 新加的变量
        //                                 that.new_arr[index].left = v.left;
        //                                 that.new_arr[index].bottom = v.bottom;
        //                                 that.new_arr[index].time = v.time;
        //                                 that.new_arr[index].stationAId = v.stationAId;
        //                                 that.new_arr[index].stationBId = v.stationBId;
        //                                 that.new_arr[index].showStationName = v.showStationName  //  新加的变量
        //                                 that.new_arr[index].locationFlag = v.locationFlag;
        //                             } else {
        //                                 let new_list = {};
        //                                 new_list.trainId = v.trainId
        //                                 new_list.trainName = v.trainName
        //                                 new_list.lineId = v.lineId
        //                                 new_list.showLineName = v.showLineName  // 新加的变量
        //                                 new_list.left = v.left 
        //                                 new_list.bottom = v.bottom
        //                                 new_list.time = v.time 
        //                                 new_list.stationAId = v.stationAId 
        //                                 new_list.showStationName = v.showStationName  //  新加的变量
        //                                 new_list.stationBId = v.stationBId
        //                                 new_list.locationFlag = v.locationFlag 
        //                                 that.arr2.push(v.trainId);
        //                                 that.new_arr.push(new_list);
        //                             }
        //                         }
        //                     })
        //                     that.positions = that.new_arr;
        //                     // console.log("that.positions:",that.positions);
        //                     that.setPositions(); //计算小车的具体位置信息

        //                 }else{
        //                     //socket 返回对个列车的情况
        //                     returnArr1.map(v=>{
        //                         v.left = -8, v.bottom = -8, v.flag = true, v.showLineName = null, v.showStationName = null

        //                         if( that.new_arr.length==0){
        //                             let new_list = {};
        //                             new_list.trainId = v.trainId
        //                             new_list.trainName = v.trainName
        //                             new_list.lineId = v.lineId
        //                             new_list.showLineName = v.showLineName  // 新加的变量
        //                             new_list.left = v.left 
        //                             new_list.bottom = v.bottom
        //                             new_list.time = v.time 
        //                             new_list.stationAId = v.stationAId 
        //                             new_list.showStationName = v.showStationName  //  新加的变量
        //                             new_list.stationBId = v.stationBId
        //                             new_list.locationFlag = v.locationFlag 
        //                             that.new_arr.push(new_list);
        //                             that.arr2.push(v.trainId);
        //                         }else{
        //                             if( that.arr2.indexOf(v.trainId)>-1){
        //                                 var index= that.arr2.indexOf(v.trainId)
        //                                 that.new_arr[index].lineId = v.lineId;
        //                                 that.new_arr[index].trainName = v.trainName;
        //                                 that.new_arr[index].showLineName = v.showLineName // 新加的变量
        //                                 that.new_arr[index].left = v.left;
        //                                 that.new_arr[index].bottom = v.bottom;
        //                                 that.new_arr[index].time = v.time;
        //                                 that.new_arr[index].stationAId = v.stationAId;
        //                                 that.new_arr[index].stationBId = v.stationBId;
        //                                 that.new_arr[index].showStationName = v.showStationName  //  新加的变量
        //                                 that.new_arr[index].locationFlag = v.locationFlag;
        //                             }else {
        //                                 let new_list = {};
        //                                 new_list.trainId = v.trainId
        //                                 new_list.trainName = v.trainName
        //                                 new_list.lineId = v.lineId
        //                                 new_list.showLineName = v.showLineName  // 新加的变量
        //                                 new_list.left = v.left 
        //                                 new_list.bottom = v.bottom
        //                                 new_list.time = v.time 
        //                                 new_list.stationAId = v.stationAId 
        //                                 new_list.showStationName = v.showStationName  //  新加的变量
        //                                 new_list.stationBId = v.stationBId
        //                                 new_list.locationFlag = v.locationFlag 
        //                                 that.arr2.push(v.trainId);
        //                                 that.new_arr.push(new_list);
        //                             }
        //                         }

        //                     })

        //                     that.positions = that.new_arr;
        //                     // console.log("that.positions:",that.positions);
        //                     that.setPositions(); //计算小车的具体位置信息
        //                 }
        //             };
        //             //关闭事件
        //             this.socket[i].onclose = function(close) {
        //                 // console.log("websocket close");
        //             };
        //         } catch (ex) {
        //             log(ex);
        //         }
        //     } else {
        //         // 浏览器不支持 WebSocket
        //         this.$alert("您的浏览器不支持Websocket通信协议，请使用Chrome或者Firefox浏览器！","温馨提示：");
        //     }
        // },
    },
    created(){
        this.all = allDate;
        console.log("date:",allDate);
    },
    mounted(){
        this.boxSelection(); //鼠标事件
    },
    destroyed(){
        // //页面销毁时关闭长连接
        // if(this.socket !== null){
        //     for( let i=0; i<this.socket.length; i++){
        //         this.socket[i].onclose(); //关闭websocket
        //     }
        // }
        //销毁
        this.$destroy();
    },
}
</script>

<style type="text/css">
.select span {
    color: red;
    font-weight: bold;
}
#trainSelected{
    position: relative;
}
/* trainSelected */
.tempDiv{
    /* border:3px dashed yellowgreen; */
    position:absolute;
    width:0px;
    height:0px;
    /* opacity:0.5; */
    /* cursor: move; */
}
/* trainSelected */
/* /鼠标移动时禁止选中文字/ */
div{
    -moz-user-select: none;
    -khtml-user-select: none;
    user-select: none;
    /* cursor: pointer; */
}
.train{
    margin-top: 20px;
    /* position: relative;  */
    z-index: 9;
}
.Train_contains{
    width:100%;
    height: 200px;
    /* overflow-x: scroll; */
    /* border: 1px solid #ccc; */
    border-radius: 4px;
    white-space:nowrap; /* 不换行 */
}
/* 线路 --标题 */
.lines .lines_title{
    margin-bottom:10px;
    margin-top: 20px;
    font-size:16px;
    color:#000;
    font-weight:bold;
}
/* 每条线的所有车站 */
.lines .stations {
    margin-top: 20px;
    display: inline-block;
    height: 234px;
    /* width: 100px; */
}
/* p:nth-child(even){} //偶数行 */
.lines .stations:nth-child(even) .station_item {
    position: relative;
    top: 100px;
    left:0;
}
.lines .stations .station_item{
    display: inline-block;
    font-size: 12px;
    margin-bottom: 20px;
    word-wrap: break-word;
    word-break: break-all;
    overflow: hidden;
}
/* 小圆圈---以及底部线 */
.lines .stations .bottom {
    position: relative;
    /* width: 100px; */
    border-bottom: 2px solid #000;
    /* border-radius: 4px; */
    margin-top: 15px;
    margin-left:10px;
}
.lines .stations .bottom .bottom1{
    position: relative;
    top: -2px;
    border-bottom: 2px solid green;
    z-index: 0;
}
/* .lines .stations .bottom .center {
    position: absolute;
    right: 35px;
    bottom: -5px;
    content: '';
    display: inline-block;
    width: 12px;
    height: 12px;
    border-radius: 50%;
    background-color: #0089ff;
    opacity: 1;
    z-index: 9;
} */
.lines .stations .bottom .circle {
    position: absolute;
    left: -11px;
    bottom: -8px;
    content: '';
    display: inline-block;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background-color: #0089ff;
    opacity: 1;
}
.lines .stations .bottom .circle:hover {
  cursor: pointer;
}
.stations:nth-last-child(1) .bottom {
  border-width:0; 
  height: 2px;
}
.stations:nth-last-child(1) .bottom .bottom1{
  border-width:0;
  height: 4px; 
}
/* .stations:nth-last-child(2) .bottom .circle1{
    position: absolute;
    right: -20px;
    bottom: -8px;
    content: '';
    display: inline-block;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    background-color: #0089ff;
    opacity: 1;
    z-index: 9;
} */
.lines .bottom::after :hover {
    bottom: -8px;
    width: 20px;
    height: 20px;
    opacity: 1;
    cursor: pointer;
}
/* 小车 */
.positions {
    width: 100%;
    position: relative;
    top: -120px;
}
.positions #actpoint {
    position: absolute;
    bottom: 20px;
    left: -6px;
    width: 50px;
    height: 22px;
    /* background: url('../../../../assets/images/tain1.png') no-repeat; */
    cursor: pointer;
}
.bg1{
    position: absolute;
    bottom: 20px;
    left: -6px;
    width: 50px;
    height: 22px; 
    background: url('../../../../assets/images/tain1.png') no-repeat;
    cursor: pointer;
}
.bg2{
    position: absolute;
    bottom: 20px;
    left: -6px;
    width: 50px;
    height: 22px;
    background: url('../../../../assets/images/tain2.png') no-repeat;
    cursor: pointer;
}
/* 小车到站后变绿色 */
.lines .stations .bottom .circle.actived {
    background-color: #3CB371 !important;
}
</style>