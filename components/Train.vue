<template>
    <div style="width: 100%;">
        <div style="display: flex;align-items: center;justify-content: space-between; height:50px;width:100vw;background-color:#3883fa;color: aliceblue;position: absolute;left:0;top:0;">
            <span style="font-family: 'Lucida Sans';font-weight: bolder;font-size: x-large;margin: 0 10px;">
            JSON TRAIN
            </span>
            <el-link target="_blank" type="success"  href="https://github.com/ATUFO/JsonTrain" style="float: right;margin-right: 20px;color: aliceblue;">
                github
            </el-link>
        </div>
        <div class="vje_container" style="margin-top:50px;">
            <div class="vje_src">
                <vue-json-editor style="height: 100%;" v-model="resourceJson" :showBtns="false" :mode="'code'"
                    @json-change="onJsonChange" @json-save="onJsonSave" @has-error="onError" />
            </div>
            <div class="vje_ex" v-if="exJson != undefined">
                <vue-json-editor height="100" style="height: 100%;" v-model="exJson" :showBtns="false" :mode="'code'"
                    @json-change="onJsonChange" @json-save="onJsonSave" @has-error="onError" />
            </div>
        </div>
        <br>
        <div>
            <el-tabs v-model="tab" type="card">
                <el-tab-pane label="JsonPath" name="jsonpath">
                    <el-input v-model="xpath" @input="onPathChange" type="text">
                        <template slot="prepend">$</template>
                    </el-input>
                </el-tab-pane>
                <el-tab-pane label="JS" name="js">
                    <el-autocomplete v-model="jst" @input="onJstChange" :fetch-suggestions="querySearch"  @focus="handleFocus" @select="handleSelect" type="text" style="width: 100%;">
                        <template slot="prepend">data</template>
                        <template slot-scope="{ item }">{{ item.word }}</template>
                    </el-autocomplete>
                </el-tab-pane>
            </el-tabs>
        </div>
    </div>
</template>
<script>
import vueJsonEditor from 'vue-json-editor'
import jp from "jsonpath"
import utils from "./utils"

export default {
    name: "Train",
    components: { vueJsonEditor },
    data() {
        return {
            tab:'jsonpath',
            resourceJson: {},
            xpath: "",
            jst:"",
            exJson: undefined
        }
    },

    completeWord: {},
    srcElement : "",

    methods: {
        onJsonSave() { },
        onJsonChange(val) {
            console.log(val);
            this.completeWord={}

            var deepVisit = (key,node)=>{
                if(typeof node === "object"){
                    if(this.completeWord[key]==null){
                        this.completeWord[key]=new Set()
                    }
                    
                    for( var i in node ) {  
                        if(node instanceof Array){
                            deepVisit(key, node[i])
                        }else{
                            this.completeWord[key].add(i)
                            deepVisit(i,node[i])
                        }
                    }
                }   
            }
            deepVisit("root",val)
            console.log(this.completeWord)
        },
        onJstChange() {
            console.log(this.xpath);
            try {
                var data = this.resourceJson
                var result = eval("data"+this.jst)
                if(typeof(result) === 'function'){
                    this.exJson = undefined
                }else{
                    this.exJson = result
                }
                // console.log(result);
            } catch (error) {
                this.exJson = undefined
            }
        },
        onPathChange() {
            console.log(this.xpath);

            this.showEx = this.xpath !== ''
            var p = "$" + this.xpath
            try {
                this.exJson = jp.query(this.resourceJson, p)
            } catch (error) {
                this.exJson = undefined
            }
        },
        onError() { },
        handleFocus(e){
            this.srcElement = e.srcElement
        },
        cursorIndex(){
            return this.srcElement?.selectionStart ? this.srcElement.selectionStart : 0
        },
        querySearch(queryString, cb,e) {
            let key = ""
            let matchStr = ""
            let dot = 0

            for (let i = this.cursorIndex(); i >= 0; i--) {
                if(this.jst[i]=='.' || this.jst[i]==' '){
                    matchStr = this.jst.substring(i+1,this.cursorIndex())
                    dot = i
                    break
                }
            }   
            if(dot != 0){
                for (let i = dot-1; i >= 0; i--) {
                    if(this.jst[i]=='.' || this.jst[i]==' '){
                        key = this.jst.substring(i+1,dot)
                        break
                    }else if(i==0){
                        key = this.jst.substring(i,dot)
                    }
                }  
            }else{
                key = "root"
                matchStr=this.jst
            }
                   

            console.log(key,matchStr,dot);
            var results = this.completeWord?.[key]
            if(results){
                results = Array.from(results).map(word=>({value: this.jst.substring(0,dot+1)+word, word:word,distance:utils.editDistance(word,matchStr)}))
                results = results.sort((a,b)=>a.distance-b.distance)
            }else{
                results = []
            }


            cb(results);
        },

        handleSelect(item){
            
        }
    }

}
</script>
<style>
.vje_container {
    display: flex;
    width: 100%;
    height: 70vh;
    justify-content: space-around;
    align-items: stretch;
}

.vje_ex,
.vje_src {
    width: 100%;
    height: 100%;
    margin: 10px;
}

.jsoneditor-vue {
    height: 100%;
}
</style>