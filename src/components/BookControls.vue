<template>
    <div class="stylebotones">
        <b-button @click="sortByAuthor">Ordenar por Autor</b-button>
        <b-button @click="sortByDate">Ordenar por fecha</b-button>
    </div>
</template>

<script>
    export default{
        methods:{
            sortByAuthor(){
                this.$emit('sortByAuthor');
            },
            sortByDate(){
                this.$emit('sortByDate');
            }
        }
    }
</script>

<style>
.stylebotones{
    justify-content: space-evenly;
    display: flex;
}
</style>