<template>
    <div class="page">
        <div>
            <inside
                v-for="o in this.ele_count"
                :key="o"
                class="card"
                :floor_count="floor_count"
                :ele_id="o"
                :press_floor="press_floor"
                :up="up"
                :down="down"
                :call_ele_id="call_ele_id"
                @childStatus="getChildStatus"
                ref="insidechild"
            ></inside>
        </div>
        <div>
            <outside
                class="floor"
                v-for="o in this.floor_count"
                :key="o"
                :floor_id="o"
                @childCallFloorUp="callFloorUp"
                @childCallFloorDown="callFloorDown"
            ></outside>
        </div>
    </div>
</template>

<script>
import Inside from "./components/inside"
import Outside from "./components/outside"
export default {
    name: "Home",
    components: {
        Inside,
        Outside
    },
    data() {
        return {
            ele_count: 5, //电梯数量,对应电梯id
            floor_count: 20, //楼层数量
            running: [], //电梯运行状态
            going_up: [], //电梯方向
            current_floor: [], //每个电梯的当前楼层
            call: [[]], //每个电梯的等待队列
            //传给子组件的值
            //要维护一个变量 保证楼层被按下时候能被监听
            press_floor: null,
            call_ele_id: 1, //要放入的ele_id
            up: false, //⬆上被按下
            down: false //下被按下
        }
    },
    methods: {
        // 计算已排序的队列[n]中a和b之间的数目计数
        // a > b or a < b is ok
        betweenCount(a, b, id) {
            if (a > b) {
                var temp = a
                a = b
                b = temp
            }
            console.log(this.call[id])
            var betweenNum = []
            for (var i in this.call[id]) {
                if (
                    this.call[id][i] <= b &&
                    this.call[id][i] >= a &&
                    betweenNum.indexOf(this.call[id][i] < 0)
                ) {
                    betweenNum.push(this.call[id][i])
                }
            }
            console.log("between:", betweenNum.length)
            return betweenNum.length
        }, //找最小最大值
        getMinInQueue(arr) {
            // if (length.arr) {
            return arr[0]
            // }
        },
        getMaxInQueue(arr) {
            // if (length.arr) {
            return arr[arr.length - 1]
            // }
        },
        //计算应该放入哪个电梯
        calculateUpDistance(press_floor) {
            var maxDistance = 2 * this.floor_count //可能出现的最小距离
            var distance = 0
            var elevatorToPush = 1 //放入的电梯序号
            // 希望在最短时间内达到呼层
            // 计算到呼层的步数（1s/步）+5s*当中停下的楼层：
            // if没有运动 else：
            // if当前层在呼层的下面或当前层
            //      电梯向上运动
            //      电梯向下运动
            // else当前层在呼层的上面
            //      电梯向上运动
            //      电梯向下运动
            for (var i = 0; i < this.ele_count; i++) {
                if (!this.running[i]) {
                    distance = Math.abs(press_floor - this.current_floor[i])
                } else {
                    var minInQueue = this.getMinInQueue(this.call[i]) //找出呼叫队列中最小的
                    var maxInQueue = this.getMaxInQueue(this.call[i]) //找出呼叫队列中最大的
                    if (this.current_floor[i] <= press_floor) {
                        if (this.going_up[i]) {
                            console.log("电梯向上，按键比当前高")
                            distance =
                                press_floor -
                                this.current_floor[i] +
                                5 *
                                    this.betweenCount(
                                        press_floor,
                                        this.current_floor[i],
                                        i
                                    )
                        } else {
                            console.log("电梯向下，按键比当前高")
                            distance =
                                this.current_floor[i] -
                                minInQueue +
                                press_floor -
                                minInQueue +
                                5 *
                                    this.betweenCount(
                                        minInQueue,
                                        press_floor,
                                        i
                                    )
                        }
                    } else {
                        if (this.going_up[i]) {
                            console.log("电梯向上，按键比当前低")
                            distance =
                                maxInQueue -
                                this.current_floor[i] +
                                maxInQueue -
                                minInQueue +
                                Math.abs(minInQueue - press_floor) +
                                5 *
                                    this.betweenCount(
                                        maxDistance,
                                        maxInQueue,
                                        i
                                    )
                            console.log(
                                "distance:",
                                distance,
                                "max:",
                                maxDistance
                            )
                        } else {
                            console.log("电梯向下，按键比当前低")
                            distance =
                                this.current_floor[i] -
                                minInQueue +
                                Math.abs(minInQueue - press_floor) +
                                5 *
                                    betweenCount(
                                        this.current_floor[i],
                                        minInQueue,
                                        i
                                    )
                        }
                    }
                    if (distance < maxDistance) {
                        maxDistance = distance
                        elevatorToPush = i + 1
                    }
                }
            }
            console.log(
                press_floor + "上楼将要放入第" + elevatorToPush + "电梯"
            )
            this.up = true
            return elevatorToPush //得到按下按键的楼层即将放入哪个电梯
        },
        calculateDownDistance(press_floor) {
            var maxDistance = 2 * this.floor_count //可能出现的最小距离
            var distance = 0
            var elevatorToPush = 1 //放入的电梯序号
            // 希望在最短时间内达到呼层
            // 计算到呼层的步数（1s/步）+5s*当中停下的楼层：
            // if没有运动 else：
            // if当前层在呼层的下面或当前层
            //      电梯向上运动
            //      电梯向下运动
            // else当前层在呼层的上面
            //      电梯向上运动
            //      电梯向下运动
            for (var i = 0; i < this.ele_count; i++) {
                if (!this.running[i]) {
                    distance = Math.abs(press_floor - this.current_floor[i])
                } else {
                    var minInQueue = this.getMinInQueue(this.call[i]) //找出呼叫队列中最小的
                    var maxInQueue = this.getMaxInQueue(this.call[i]) //找出呼叫队列中最大的
                    if (this.current_floor[i] <= press_floor) {
                        if (this.going_up[i]) {
                            console.log("电梯向上，按键比当前高")
                            distance =
                                maxInQueue -
                                this.current_floor[i] +
                                Math.abs(maxInQueue - press_floor) +
                                5 *
                                    this.betweenCount(
                                        this.current_floor[i],
                                        maxInQueue,
                                        i
                                    )
                        } else {
                            console.log("电梯向下，按键比当前高")
                            distance =
                                this.current_floor[i] -
                                minInQueue +
                                maxInQueue -
                                minInQueue +
                                Math.abs(maxInQueue - press_floor) +
                                5 *
                                    this.betweenCount(
                                        maxDistance,
                                        maxInQueue,
                                        i
                                    )
                        }
                    } else {
                        if (this.going_up[i]) {
                            console.log("电梯向上，按键比当前低")
                            distance =
                                maxInQueue -
                                this.current_floor[i] +
                                maxInQueue -
                                press_floor +
                                5 *
                                    this.betweenCount(
                                        maxInQueue,
                                        press_floor,
                                        i
                                    )
                            console.log(
                                "distance:",
                                distance,
                                "max:",
                                maxDistance
                            )
                        } else {
                            console.log("电梯向下，按键比当前低")
                            distance =
                                this.current_floor[i] -
                                press_floor +
                                5 *
                                    betweenCount(
                                        press_floor,
                                        this.current_floor[i],
                                        i
                                    )
                        }
                    }
                    if (distance < maxDistance) {
                        maxDistance = distance
                        elevatorToPush = i + 1
                    }
                }
            }
            console.log(
                press_floor + "下楼将要放入第" + elevatorToPush + "电梯"
            )
            this.down = true
            return elevatorToPush //得到按下按键的楼层即将放入哪个电梯
        },
        //得到当前哪个楼层被外部呼叫,把这个呼叫信号传递给最近的电梯
        callFloorUp(floor_num) {
            this.press_floor = floor_num
            //把当前需要加入的电梯ID、楼层以及上下都传给子组件
            this.call_ele_id = this.calculateUpDistance(floor_num)

            //把状态传递给响应电梯子组件
            this.$refs.insidechild[this.call_ele_id - 1].addOutside(
                this.up,
                this.down,
                floor_num
            )
            console.log(
                "传入第几个电梯：",
                this.call_ele_id,
                "第几层楼",
                this.press_floor
            )

            for (var i = 0; i < this.ele_count; i++) {
                console.log("call:", this.call[i])
            }
        },
        callFloorDown(floor_num) {
            this.press_floor = floor_num
            this.call_ele_id = this.calculateDownDistance(floor_num)
            this.$refs.insidechild[this.call_ele_id - 1].addOutside(
                this.up,
                this.down,
                floor_num
            )
            //测试log
            console.log(
                "传入第几个电梯：",
                this.call_ele_id,
                "第几层楼",
                this.press_floor
            )
            for (var i = 0; i < this.ele_count; i++) {
                console.log("call:", this.call[i])
            }
        },
        //从inside获取每个电梯的状态
        getChildStatus(
            id,
            child_current,
            child_running,
            child_queue,
            child_up
        ) {
            this.current_floor[id - 1] = child_current
            this.running[id - 1] = child_running
            this.call[id - 1] = child_queue
            this.going_up[id - 1] = child_up
            console.log("ele_id:", id)
        }
    },
    created() {
        //初始化电梯状态
        for (var i = 0; i < this.ele_count; ++i) {
            this.running[i] = false
            this.current_floor[i] = 1
        }
    }
}
</script>

<style>
.card {
    display: inline-flex;
    width: 370px;
    margin: 20px auto 20px auto;
}
.floor {
    display: inline-flex;
    justify-content: space-around;
    width: 370px;
}
</style>
