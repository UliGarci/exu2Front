<template>
  <div class="styleactions">
    <b-row>
        <b-row class="mt-5"><b-button variant="success" class="styleb" @click="insert"><b-icon icon="plus-square" aria-hidden="true"></b-icon></b-button></b-row>
        <b-row class="mt-5 styledrag"><b-button variant="primary" class="styleb" @dragover.prevent @drop="edit"><b-icon icon="pencil-square" aria-hidden="true"></b-icon></b-button></b-row>
        <b-row class="mt-5 styledrag"><b-button variant="danger" class="styleb" @dragover.prevent @drop="deleteBook"><b-icon icon="trash" aria-hidden="true"></b-icon></b-button></b-row>  
    </b-row>
    
  </div>
</template>

<script>
export default {
    methods:{
        edit(){
            this.$emit('edit');
        },
        deleteBook(){
            this.$emit('deleteBook')
        },
        insert(){
            this.$emit('insert');
        }
    }

}
</script>

<style>
.styleactions{
    justify-content: space-around;
    display: flex;
}

.styleb{
    height: 20vh;
}

.styledrag{
    border: 3px dashed black;
}
</style>