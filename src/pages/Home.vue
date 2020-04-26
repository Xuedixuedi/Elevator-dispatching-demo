<template>
    <div class="page">
        <inside v-for="o in 5" :key="o" class="card"></inside>
        <outside></outside>
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
    }
}
</script>

<style>
.card {
    display: inline-flex;
    width: 370px;
    margin: 20px auto 20px auto;
}
</style>
