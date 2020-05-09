<template>
    <div class="page">
        <div>
            <inside
                v-for="o in this.ele_count"
                :key="o"
                class="card"
                :floor_count="floor_count"
                :ele_id="o"
                @childStatus="getChildStatus"
            ></inside>
        </div>
        <div>
            <outside
                class="floor"
                v-for="o in this.floor_count"
                :key="o"
                :floor_id="o"
                @childCallFloor="callFloor"
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
            floor_call: [], //当前呼叫且尚未分配电梯的floor,数组表示队列
            running: [], //电梯运行状态
            going_up: [], //电梯方向
            current_floor: [], //每个电梯的当前楼层
            call: [[]] //每个电梯的等待队列
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
        calculateDistance(press_floor) {
            var maxDistance = 2 * this.floor_count //可能出现的最小距离
            var distance = 0
            var elevatorToPush = 0 //放入的电梯序号
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
                        elevatorToPush = i
                    }
                }
            }
            console.log("将要放入第" + i + "电梯")
        },
        //得到当前哪个楼层被外部呼叫,把这个呼叫信号传递给最近的电梯
        callFloor(floor_num) {
            //入队
            this.calculateDistance(floor_num)
            // this.floor_call.push(floor_num)
            console.log("floorcall:" + this.floor_call)
            //测试log
            console.log(
                "running:" + this.running + "curr:" + this.current_floor
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
