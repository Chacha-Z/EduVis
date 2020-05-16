<template>
    <div class='title-bar'>
        <slot>系统视图</slot>
    </div>
    
</template>

<script>
export default {
    name: 'title-bar',
    data(){
        return {

        }
    },
    methods: {
        
    },
    mounted(){
    }
}

</script>

<style scoped="scoped">
    .title-bar {
        font-size: 12px;
        height: 30px;
        line-height: 30px;
        background-color: rgb(252, 252, 252);
        border-bottom: rgb(245, 237, 237) 1px solid;
        padding-left: 10px;
    }
        
</style>