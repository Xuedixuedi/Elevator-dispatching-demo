<template>
    <div>
        <div class="floor" v-for="o in 20" :key="o">
            <div>
                <el-tag class="number-tag">{{ o }}</el-tag>
                <el-button-group>
                    <el-button size="small" type="primary"
                        ><i class="el-icon-top"></i
                    ></el-button>
                    <el-button size="small" type="primary"
                        ><i class="el-icon-bottom"></i
                    ></el-button>
                </el-button-group>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: "Outside"
}
</script>

<style>
.floor {
    display: inline-flex;
    justify-content: space-around;
    width: 370px;
}

.number-tag {
    width: 50px;
    text-align: center;
    margin: 20px auto 20px auto;
}
</style>
