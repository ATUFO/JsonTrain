<template>
    <div>
        <div>
            <div class="text-input">
                <vue-json-editor style="height: 100%;" v-model="textb" :showBtns="false" :mode="'code'"
                    @json-change="onJsonChange" @json-save="onJsonSave" @has-error="onError" />
                    <!-- <el-input type="textarea" v-model="textb"></el-input> -->
            </div>
   
            <div class="text-input">
                <vue-json-editor style="height: 100%;" v-model="texta" :showBtns="false" :mode="'code'"
                    @json-change="onJsonChange" @json-save="onJsonSave" @has-error="onError" />
                    <!-- <el-input type="textarea" v-model="texta"></el-input> -->
            </div>
        </div>
        <div>
            <!-- <el-button @click="handleSort">Sort</el-button>
            <el-button @click="handleDiff">Diff</el-button> -->
            <el-button @click="handleXDiff">X Diff</el-button>
            <el-checkbox v-model="sort">排序</el-checkbox>
            <el-checkbox v-model="full">全部</el-checkbox>    
        </div>
        <pre  v-highlightjs="xDiffResult"><code class="diff" style="height: 50vh;"></code></pre>
        <!-- <code-diff :old-string="oldStr" :new-string="newStr" :context="10" :outputFormat="outputFormat" /> -->
        
    </div>
</template>
<script>

import CodeDiff from 'vue-code-diff'
import _ from "loadsh"
import jsonDiff from "json-diff"
import vueJsonEditor from 'vue-json-editor'

export default {
    components: { CodeDiff, vueJsonEditor },
    data() {
        return {
            texta: '',
            textb: '',
            oldStr:'',
            newStr:'',
            outputFormat: 'side-by-side',
            xDiffResult:'',
            full:false,
            sort:true
        }
    }
    , methods: {
        handleDiff(){
            this.oldStr =JSON.stringify(this.textb,null,2)
            this.newStr =JSON.stringify(this.texta,null,2)
            // console.log("diff");
            // console.log(this.oldStr);
            // console.log(this.newStr);
        },
        handleSort(){
            this.texta = _.sortBy(this.texta)
            this.textb= _.sortBy(this.textb)
        },
        handleXDiff(){
            console.log("xdiff");
            this.xDiffResult=jsonDiff.diffString(this.textb,this.texta,{full:this.full,sort:this.sort})
        },
        onJsonChange() { },
        onJsonSave() { },
        onError() { }
    }
}
</script>
<style>
.text-input{
    width: 49%;
    height: 40vh;
    display: inline-block;
}
.jsoneditor-vue {
    height: 100%;
}
</style>
