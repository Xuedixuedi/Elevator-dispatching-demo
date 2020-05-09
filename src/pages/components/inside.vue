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
            outside: [], //外部电梯请求队列
            inside: [], //内部呼梯信号
            call: [] //存放内外所有呼梯层
        }
    },
    props: {
        ele_id: Number, //电梯编号
        floor_count: Number //总楼层数
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
            this.$emit(
                "childStatus",
                this.ele_id,
                this.current_floor,
                this.running,
                this.call,
                this.going_up
            )
            // console.log(
            //     "Up:",
            //     this.going_up,
            //     "current_floor:",
            //     this.current_floor,
            //     "call:",
            //     this.call
            // )
        },
        //内部按钮呼梯
        handleInsideButtonClick(floor) {
            this.button_click[floor] = true
            //o是呼梯楼层,把内部呼梯信号改成1，加入呼梯队列
            this.inside[floor] = 1
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
            this.call.push(floor)
            this.call.sort()
            if (!this.running) {
                this.checkStatus()
            }
            // }
        },
        //找最小最大值
        getMinInQueue(arr) {
            if (length.arr) {
                return arr[0]
            }
        },
        getMaxInQueue(arr) {
            if (length.arr) {
                return arr[arr.length - 1]
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
            if (this.running && this.door == false) {
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
                this.goingup &&
                (!this.running ||
                    this.current_floor < this.getMaxInQueue(this.call))
                    ? (this.goingup = true)
                    : (this.goingup = false)
                !this.goingup &&
                (!this.running ||
                    this.current_floor > this.getMinInQueue(this.call))
                    ? (this.goingup = false)
                    : (this.goingup = true)
            }
        },
        run() {
            if (this.running) {
                if (this.call.indexOf(this.current_floor) > -1) {
                    this.ding(this.current_floor)
                } else {
                    this.going_up ? this.moveUp() : this.moveDown()
                    this.updateFloorInfo()
                }
                this.checkStatus()
            }

            console.log("Running:", this.running)
        },
        //暂停计时器，熄灭该楼层的灯光
        ding(floor) {
            let _this = this
            let that = this
            //需要电梯停下，就把timer清空
            if (this.timer) {
                clearInterval(this.timer)
            }
            //把当前楼层移除队列
            this.removeFromQueue(this.call, floor)
            this.button_click[floor] = 0
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
        for (var i = 0; i < this.max_floor; ++i) {
            this.button_click[i] = false
            this.outside[i] = 0
            this.inside[i] = 0
            this.floor_id[i] = i + 1 //这个顺序是 21-o
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
