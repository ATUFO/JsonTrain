<template>
    <div style="width: 100%;">
        <div style="display: flex;align-items: center;justify-content: space-between; height:50px;width:100vw;background-color:#3883fa;color: aliceblue;position: absolute;left:0;top:0;">
            <span style="font-family: 'Lucida Sans';font-weight: bolder;font-size: x-large;margin: 0 10px;">
            JSON TRAIN
            </span>
            <span style="float: right;margin-right: 20px;font-size: x-small;">
                @Dov
            </span>
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
                    <el-input v-model="jst" @input="onJstChange" type="text">
                        <template slot="prepend">this.</template>
                    </el-input>
                </el-tab-pane>
            </el-tabs>
        </div>
    </div>
</template>
<script>
import vueJsonEditor from 'vue-json-editor'
import jp from "jsonpath"
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
    methods: {
        onJsonSave() { },
        onJsonChange(val) {
            console.log(val);
        },
        onJstChange() {
            console.log(this.xpath);
            try {
                var result = eval("this.resourceJson."+this.jst)
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
        onError() { }
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