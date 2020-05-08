<template>
    <div class="page">
        <div>
            <inside
                v-for="o in this.ele_count"
                :key="o"
                class="card"
                :inputFloor="floor[0]"
                :floor_count="floor_count"
                :ele_id="o"
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
            floor: [1, 2, 3, 4, 5, 6, 7, 8, 9], //楼层
            ele_count: 5, //电梯数量,对应电梯id
            floor_count: 20, //楼层数量
            floor_call: [] //当前呼叫且尚未分配电梯的floor,数组表示队列
        }
    },
    methods: {
        //得到当前哪个楼层被呼叫,把这个呼叫信号传递给最近的电梯
        callFloor(floor_num) {
            //入队
            this.floor_call.push(floor_num)
            console.log(this.floor_call)
        }
    },
    computed: {}
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
