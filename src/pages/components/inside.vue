<template>
    <div class="box">
        <el-card class="box-card">
            <el-card class="floor-num" shadow="always">
                <div>
                    <i class="el-icon-caret-top" v-show="this.going_up"></i>
                    <i
                        class="el-icon-caret-top disabled"
                        v-show="!this.going_up"
                    ></i>
                    {{ this.current_floor }}
                    <i class="el-icon-caret-bottom" v-show="!this.going_up"></i>
                    <i
                        class="el-icon-caret-bottom disabled"
                        v-show="this.going_up"
                    ></i>
                </div>
            </el-card>
            <div class="button-group">
                <div
                    v-for="o in this.floor_count"
                    :key="o"
                    class="button-place"
                >
                    <el-button
                        type="primary"
                        plain
                        class="button"
                        @click="handleInsideButtonClick(o)"
                        :class="{ 'button-click': button_click[o] }"
                    >
                        {{ o }}
                    </el-button>
                </div>
            </div>
            <div class="control">
                <el-switch
                    class="switch"
                    style="display: block"
                    v-model="door"
                    active-color="#13ce66"
                    inactive-color="#ff4949"
                    active-text="开门"
                    inactive-text="关门"
                    @click.native="doorAlert()"
                >
                </el-switch>
                <el-button
                    @click="handlePhoneClick()"
                    type="warning"
                    icon="el-icon-phone"
                    circle
                ></el-button>
                <el-button
                    @click="handleBellClick()"
                    type="danger"
                    icon="el-icon-message-solid"
                    circle
                ></el-button>
            </div>
        </el-card>
    </div>
</template>

<script>
export default {
    name: "Inside",
    data() {
        return {
            button_click: [],
            timer: null,
            running: false, //是否在运行
            going_up: true, //true向上 false向下
            door: false, //门的开关状态
            current_floor: 1, //当前楼层
            max_floor: 20,
            min_floor: 1,
            floor_id: [],
            outside_up: [], //外部电梯向上请求队列
            outside_down: [], //外部电梯向下请求队列
            inside: [], //内部呼梯信号
            call: [] //存放内外所有呼梯层
        }
    },
    props: {
        ele_id: Number, //电梯编号
        floor_count: Number, //总楼层数
        call_ele_id: Number,
        press_floor: Number, //父组件的按下楼层
        up: Boolean, //是向上
        down: Boolean
    },
    methods: {
        handlePhoneClick() {
            alert("☎️请求通话！☎️")
        },
        handleBellClick() {
            alert("🆘请求支援!🆘")
        },
        //更新上下楼的时候的内部指示灯
        updateFloorInfo() {
            //每秒向父组件传递该电梯当前的id和楼层
            //接受父组件传值回来
            this.$emit(
                "childStatus",
                this.ele_id,
                this.current_floor,
                this.running,
                this.call,
                this.going_up
            )
        },
        //更新outside
        updateOutsideUp(floor) {
            this.$emit("updateOutsideUp", floor)
        },
        updateOutsideDown(floor) {
            this.$emit("updateOutsideDown", floor)
        },
        //内部按钮呼梯
        handleInsideButtonClick(floor) {
            this.button_click[floor] = true
            //floor是呼梯楼层,把内部呼梯信号改成1，加入呼梯队列
            this.inside[floor] = 1
            console.log("内部按键对列我康康", this.inside)
            this.dial(floor)
        },
        //外部button click
        outsideButtonClick(floor) {
            this.outside[floor] = 1
            this.dial(floor)
        },
        //blog里面说，用一个按钮函数就可以
        dial(floor) {
            // if (this.call.indexOf(floor) < 0) {
            console.log("现在按下第几层？", floor)
            this.call.push(floor)
            this.call.sort()
            console.log("现在的队列是：", this.call)
            if (!this.running) {
                this.checkStatus()
            }
            // }
        },
        //找最小最大值
        getMinInQueue(arr) {
            if (arr.length <= 0) {
                console.log("找不到最小值")
                return false
            }
            if (arr.length == 1) {
                return arr[0]
            } else {
                var min = arr[0]
                for (var i in arr) {
                    if (arr[i] < min) {
                        min = arr[i]
                    }
                }
                return min
            }
        },
        getMaxInQueue(arr) {
            if (arr.length <= 0) {
                console.log("找不到最大值")
                return false
            }
            if (arr.length == 1) {
                return arr[0]
            } else {
                var max = arr[0]
                for (var i in arr) {
                    if (arr[i] > max) {
                        max = arr[i]
                    }
                }
                return max
            }
        },
        //在队列中删除指定楼层
        removeFromQueue(queue, floor) {
            console.log("要删除的数字是：" + floor)
            var index = queue.indexOf(floor)
            if (index > -1) {
                queue.splice(index, 1)
            }
        },
        //开关门
        openDoor() {
            this.door = true
        },
        closeDoor() {
            this.door = false
            console.log("door closed")
        },
        //开门警告
        doorAlert() {
            if (this.running) {
                alert("运行期间禁止开门！")
                this.door = false
            }
        },
        //判断状态，每隔一段时间执行
        checkStatus() {
            //判断呼叫队列是否还有东西，修改运行状态
            this.running = this.call.length > 0 ? true : false
            //是否在底层
            if (this.current_floor == this.min_floor) {
                this.going_up = true
                //是否在顶层
            } else if (this.current_floor == this.max_floor) {
                this.going_up = false
            } else {
                //在中间层的状态判断
                this.going_up &&
                (!this.running ||
                    this.current_floor <= this.getMaxInQueue(this.call))
                    ? (this.going_up = true)
                    : (this.going_up = false)
                !this.going_up &&
                (!this.running ||
                    this.current_floor >= this.getMinInQueue(this.call))
                    ? (this.going_up = false)
                    : (this.going_up = true)
                console.log(this.getMaxInQueue(this.call), this.going_up)
            }
        },
        //把外面的加进来
        addOutside(up, down, press_floor) {
            console.log("父组件触发我了,我是电梯", this.ele_id)
            console.log("现在按下第几层？", press_floor)
            if (up == true) {
                console.log("按下了向上按键")
                this.outside_up[press_floor] = 1
            } else if (down == true) {
                console.log("按下了向下按键")
                this.outside_down[press_floor] = 1
            }
            this.dial(press_floor)
        },
        run() {
            var need_stop = false
            if (this.running) {
                if (this.call.indexOf(this.current_floor) > -1) {
                    //到达内部呼叫楼层
                    if (this.inside[this.current_floor] == 1) {
                        this.removeFromQueue(this.call, this.current_floor)
                        this.inside[this.current_floor] = 0
                        this.button_click[this.current_floor] = 0
                        need_stop = true
                    }
                    if (this.going_up) {
                        if (this.outside_up[this.current_floor] == 1) {
                            console.log("到达外面呼叫：", this.current_floor)
                            this.removeFromQueue(this.call, this.current_floor)
                            this.outside_up[this.current_floor] = 0
                            this.updateOutsideUp(this.current_floor)
                            need_stop = true
                        }
                        if (
                            this.outside_down[this.current_floor] == 1 &&
                            this.current_floor == this.getMaxInQueue(this.call)
                        ) {
                            this.removeFromQueue(this.call, this.current_floor)
                            this.outside_down[this.current_floor] = 0
                            this.updateOutsideDown(this.current_floor)
                            need_stop = true
                        }
                    } else {
                        if (this.outside_down[this.current_floor] == 1) {
                            this.removeFromQueue(this.call, this.current_floor)
                            this.outside_down[this.current_floor] = 0
                            this.updateOutsideDown(this.current_floor)
                            need_stop = true
                        }
                        if (
                            this.outside_up[this.current_floor] == 1 &&
                            this.current_floor == this.getMinInQueue(this.call)
                        ) {
                            this.removeFromQueue(this.call, this.current_floor)
                            this.outside_up[this.current_floor] = 0
                            this.updateOutsideUp(this.current_floor)
                            need_stop = true
                        }
                    }
                    if (need_stop) {
                        console.log("stop:", this.current_floor)
                        this.ding(this.current_floor)
                    } else {
                        this.going_up ? this.moveUp() : this.moveDown()
                    }
                } else {
                    this.going_up ? this.moveUp() : this.moveDown()
                    this.updateFloorInfo()
                }
                this.checkStatus()
            }
        },
        //暂停计时器，熄灭该楼层的灯光
        ding(floor) {
            let _this = this
            let that = this
            //需要电梯停下，就把timer清空
            if (this.timer) {
                clearInterval(this.timer)
            }
            this.openDoor()
            //不会重复执行的延时函数
            setTimeout(function() {
                _this.closeDoor()
                setTimeout(function() {
                    that.timer = setInterval(that.run, 1000)
                }, 3000)
            }, 4000)
        },
        //上行函数
        moveUp() {
            if (this.current_floor < this.max_floor) {
                this.current_floor++
            }
        },
        //下行
        moveDown() {
            if (this.current_floor > this.min_floor) {
                this.current_floor--
            }
        }
    },
    created() {
        //初始化电梯状态
        for (var i = 0; i <= this.max_floor; ++i) {
            this.button_click[i] = false
            this.outside_up[i] = 0
            this.outside_down[i] = 0
            this.inside[i] = 0
            this.floor_id[i] = i //这个顺序是
        }
    },
    mounted() {
        //每秒执行一次run函数
        this.timer = setInterval(this.run, 1000)

        //下面这段代码我也看不太懂，但是实现了定期更新
        // if (this.timer) {
        //     clearInterval(this.timer)
        // } else {
        //     this.timer = setInterval(() => {
        //         // 调用相应的接口，渲染数据
        //         //他意思大概就是隔一段时间你要检测一下当前楼层，外部请求这些
        //         //这里就是检测用的！
        //         // console.log(this.call)
        //         //上行or下行
        //         if (this.running && this.going_up) {
        //             this.current_floor++
        //         } else if (this.running && !this.going_up) {
        //             this.current_floor--
        //         }
        //         this.checkStatus()
        //     }, 3000)
        // }
    },
    destroyed() {
        clearInterval(this.timer)
    }
}
</script>

<style lang="stylus" scoped>

.box-card {
    width: 340px;
    height: 800px;
    padding:10px

    .floor-num {
        background-color: #409eff;
        text-align: center;
        font-size: 150%
        color: #ffffff;
        margin-bottom: 20px;

        .el-icon-caret-top{
            margin-right:40px
        }

        .el-icon-caret-bottom{
            margin-left:40px
        }

        .disabled{
            color: #152959
        }

    }

    .button-group{
        width: 100%
        overflow: hidden;
        height: 0
        padding-bottom: 200%
        display: flex;
        flex-wrap: wrap;

        .button-place{
            width: 50%
            display: flex;
            justify-content: center;
            margin-bottom: 20px

            .button{
                width:120px
            }
            .button-click{
                color:#ffffff
                background: #409eff
            }
        }

    }

    .control{
        display: flex;
        justify-content: space-around;
        align-items: center;

        .switch{
            margin-left: 20px;
        }
    }


}
</style>
