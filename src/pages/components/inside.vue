<template>
    <div class="box">
        <el-card class="box-card">
            <el-card class="floor-num" shadow="always">
                <div>
                    <i class="el-icon-caret-top"></i>
                    1
                    <i class="el-icon-caret-bottom"></i>
                </div>
            </el-card>
            <div class="button-group">
                <div v-for="o in 20" :key="o" class="button-place">
                    <el-button type="primary" plain class="button">
                        {{ 21 - o }}
                    </el-button>
                </div>
            </div>
            <div class="control">
                <el-switch
                    class="switch"
                    style="display: block"
                    v-model="value2"
                    active-color="#13ce66"
                    inactive-color="#ff4949"
                    active-text="开门"
                    inactive-text="关门"
                >
                </el-switch>
                <el-button
                    type="warning"
                    icon="el-icon-phone"
                    circle
                ></el-button>
                <el-button
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
            value2: false
        }
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
    }

    .button-group{
        // background-color:#267837
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
