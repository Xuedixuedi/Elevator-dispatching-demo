<template>
    <div>
        <div>
            <el-tag class="number-tag">{{ this.floor_id }}</el-tag>
            <el-button-group>
                <el-button
                    size="small"
                    type="primary"
                    @click="handleUpClick()"
                    :class="{ 'button-click': this.up }"
                    ><i class="el-icon-top"></i
                ></el-button>
                <el-button
                    size="small"
                    type="primary"
                    @click="handleDownClick()"
                    :class="{ 'button-click': this.down }"
                    ><i class="el-icon-bottom"></i
                ></el-button>
            </el-button-group>
        </div>
    </div>
</template>

<script>
export default {
    name: "Outside",
    props: {
        floor_id: Number //按钮对应楼层
    },
    data() {
        return {
            //当前的上下键是否已经按亮
            up: false,
            down: false
        }
    },
    methods: {
        //点击上、下按钮，把该楼层加入距离该楼层最近的电梯的等待队列中
        handleUpClick() {
            if (!this.up) {
                //把当前呼叫楼层传给父组件
                this.up = true
                this.$emit("childCallFloorUp", this.floor_id)
            }
        },
        handleDownClick() {
            if (!this.down) {
                //把当前呼叫楼层传给父组件
                this.down = true
                this.$emit("childCallFloorDown", this.floor_id)
            }
        },
        upStateRestoration() {
            console.log(this.floor_id, "变色")
            this.up = false
        },
        downStateRestoration() {
            this.down = false
        }
    }
}
</script>

<style>
.number-tag {
    width: 50px;
    text-align: center;
    margin: 20px auto 20px auto;
}

.button-click {
    background: #b5d2ef;
}
</style>
