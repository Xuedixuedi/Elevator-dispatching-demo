<template>
    <div class="page">
        <elevator1 v-for="o in 5" :key="o" class="card"></elevator1>
    </div>
</template>

<script>
import Elevator1 from "./components/elevator1"
export default {
    name: "Home",
    components: {
        Elevator1
    }
}
</script>

<style>
.page {
    /* display: flex;
    justify-content: space-between; */
}

.card {
    display: inline-flex;
    width: 370px;
    margin: 20px auto 20px auto;
}
</style>
