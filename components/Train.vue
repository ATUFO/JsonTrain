<template>
    <div class="train-page">
        <header class="app-header">
            <span class="app-title">JSON TRAIN</span>
            <nav class="app-nav">
                <nuxt-link class="nav-link" to="/">Train</nuxt-link>
                <nuxt-link class="nav-link" to="/diff">Diff</nuxt-link>
                <a class="nav-link nav-link--external" target="_blank" rel="noopener" href="https://github.com/ATUFO/JsonTrain">GitHub</a>
            </nav>
        </header>
        <main class="train-main">
            <div class="editors-row">
                <div class="editor-panel editor-panel--left">
                    <div class="editor-panel__label">输入 JSON</div>
                    <div class="editor-panel__body">
                        <vue-json-editor
                            ref="leftEditor"
                            class="json-editor"
                            v-model="resourceJson"
                            :showBtns="false"
                            :mode="'code'"
                            @json-change="onJsonChange"
                            @json-save="onJsonSave"
                            @has-error="onError"
                        />
                    </div>
                </div>
                <div class="editors-center">
                    <el-button
                        type="primary"
                        :disabled="exJson === undefined"
                        @click="applyResultToLeft"
                    >
                        结果覆盖到左侧
                    </el-button>
                </div>
                <div class="editor-panel editor-panel--right">
                    <div class="editor-panel__label">结果</div>
                    <div class="editor-panel__body">
                        <vue-json-editor
                            class="json-editor"
                            v-model="exJsonForEditor"
                            :showBtns="false"
                            :mode="'code'"
                            @json-change="onJsonChange"
                            @json-save="onJsonSave"
                            @has-error="onError"
                        />
                    </div>
                </div>
            </div>
            <div class="query-section">
                <el-tabs v-model="tab" type="card" class="query-tabs">
                    <el-tab-pane label="JsonPath" name="jsonpath">
                        <el-input v-model="xpath" @input="onPathChange" placeholder="例如: .data.list" lang="en" spellcheck="false">
                            <template slot="prepend">$</template>
                        </el-input>
                    </el-tab-pane>
                    <el-tab-pane label="JS" name="js">
                        <el-autocomplete
                            v-model="jst"
                            @input="onJstChange"
                            :fetch-suggestions="querySearch"
                            @focus="handleFocus"
                            @select="handleSelect"
                            placeholder="例如: .filter(x => x.id > 0)"
                            style="width: 100%;"
                            lang="en"
                            spellcheck="false"
                        >
                            <template slot="prepend">data</template>
                            <template slot-scope="{ item }">{{ item.word }}</template>
                        </el-autocomplete>
                    </el-tab-pane>
                </el-tabs>
            </div>
        </main>
    </div>
</template>
<script>
import vueJsonEditor from 'vue-json-editor'
import jp from "jsonpath"
import utils from "./utils"
import _ from "loadsh"
export default {
    name: "Train",
    components: { vueJsonEditor },
    data() {
        return {
            tab:'jsonpath',
            resourceJson: "",
            xpath: "",
            jst:"",
            exJson: undefined
        }
    },

    completeWord: {},
    srcElement : "",

    computed: {
        exJsonForEditor: {
            get() {
                return this.exJson !== undefined ? this.exJson : {}
            },
            set(val) {
                this.exJson = val
            }
        }
    },

    mounted() {
        // 清空编辑器内容
        this.$nextTick(() => {
            if (this.$refs.leftEditor && this.$refs.leftEditor.editor) {
                this.$refs.leftEditor.editor.setText('')
            }
        })
    },

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
            deepVisit("data",val)
            console.log(this.completeWord)

            // 如果 jsonpath 或 js 路径不为空，则按路径取值
            if (this.tab === 'jsonpath' && this.xpath) {
                this.onPathChange()
            } else if (this.tab === 'js' && this.jst) {
                this.onJstChange()
            }
        },
        onJstChange() {
            console.log(this.xpath);
            try {
                
                var data = this.resourceJson
                var result = eval("data"+this.jst)
                if(typeof(result) === 'function'){
                    this.exJson = undefined
                } else if (typeof result === 'string') {
                    try {
                        const parsed = JSON.parse(result)
                        if (parsed !== null && typeof parsed === 'object') {
                            this.exJson = parsed
                        } else {
                            this.exJson = result
                        }
                    } catch (e) {
                        this.exJson = result
                    }
                } else {
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
                let result = jp.query(this.resourceJson, p)
                // 若仅有一个结果，且为对象或可解析为对象的字符串，则以对象形式展示在右侧
                if (Array.isArray(result) && result.length === 1) {
                    const single = result[0]
                    if (single !== null && typeof single === 'object' && !Array.isArray(single)) {
                        this.exJson = single
                    } else if (typeof single === 'string') {
                        try {
                            const parsed = JSON.parse(single)
                            if (parsed !== null && typeof parsed === 'object') {
                                this.exJson = parsed
                            } else {
                                this.exJson = result
                            }
                        } catch (e) {
                            this.exJson = result
                        }
                    } else {
                        this.exJson = result
                    }
                } else {
                    this.exJson = result
                }
            } catch (error) {
                this.exJson = undefined
            }
        },
        onError() { },
        /** 将右侧结果覆盖到左侧作为输入，并再次执行当前查询 */
        applyResultToLeft() {
            if (this.exJson === undefined) return
            this.resourceJson = JSON.parse(JSON.stringify(this.exJson))
            this.onJsonChange(this.resourceJson)
            if (this.tab === 'jsonpath') {
                this.onPathChange()
            } else {
                this.onJstChange()
            }
        },
        handleFocus(e){
            this.srcElement = e.srcElement
        },
        cursorIndex(){
            return this.srcElement?.selectionStart ? this.srcElement.selectionStart : 0
        },
        querySearch(queryString, cb,e) {
            let key = ""
            let matchStr = ""
            let dot = -1

            for (let i = this.cursorIndex()-1; i >= 0; i--) {
                if(this.jst[i]=='.' || this.jst[i]==' ' || this.jst[i]==';'){
                    matchStr = this.jst.substring(i+1,this.cursorIndex())
                    dot = i
                    break
                }
            }   
            if(dot != -1){
                if(dot!=0){
                    for (let i = dot-1; i >= 0; i--) {
                        if(this.jst[i]=='.' || this.jst[i]==' ' || this.jst[i]==';'){
                            key = this.jst.substring(i+1,dot)
                            break
                        }
                    } 
                }else{
                    key="data"
                }

                 
                
                if(key != ""){
                    console.log(key,matchStr,dot);
                    var results = this.completeWord?.[key]
                    if(results){
                        results = Array.from(results).map(word=>({value: this.jst.substring(0,dot+1)+word, word:word,distance:utils.editDistance(word,matchStr)}))
                        results = results.sort((a,b)=>a.distance-b.distance)
                    }else{
                        results = []
                    }

                    cb(results);
                }
            }
        },

        handleSelect(item){
            
        }
    }

}
</script>
<style scoped>
.train-page {
    min-height: 100vh;
    background: #f5f7fa;
}

.app-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    height: 52px;
    padding: 0 20px;
    background: linear-gradient(135deg, #3883fa 0%, #2563eb 100%);
    color: #fff;
    box-shadow: 0 2px 8px rgba(56, 131, 250, 0.25);
}

.app-title {
    font-family: 'Lucida Sans', 'Segoe UI', sans-serif;
    font-weight: 700;
    font-size: 1.25rem;
    letter-spacing: 0.02em;
}

.app-nav {
    display: flex;
    align-items: center;
    gap: 8px;
}

.nav-link {
    color: rgba(255, 255, 255, 0.9) !important;
    padding: 6px 12px;
    border-radius: 6px;
    font-size: 14px;
    text-decoration: none;
}

.nav-link:hover,
.nav-link.nuxt-link-active {
    color: #fff !important;
    background: rgba(255, 255, 255, 0.15);
}

.nav-link--external {
    margin-left: 8px;
}

.train-main {
    padding: 20px;
    max-width: 100%;
    box-sizing: border-box;
}

.editors-row {
    display: flex;
    width: 100%;
    height: 72vh;
    min-height: 320px;
    gap: 12px;
    align-items: stretch;
    margin-bottom: 16px;
}

.editor-panel {
    flex: 1;
    min-width: 0;
    display: flex;
    flex-direction: column;
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.06);
    overflow: hidden;
}

.editor-panel__label {
    padding: 10px 14px;
    font-size: 13px;
    font-weight: 600;
    color: #303133;
    background: #fafafa;
    border-bottom: 1px solid #ebeef5;
}

.editor-panel__body {
    flex: 1;
    min-height: 0;
    padding: 8px;
}

.json-editor {
    height: 100%;
}

.json-editor >>> .jsoneditor-vue {
    height: 100%;
    border: 1px solid #e4e7ed;
    border-radius: 6px;
}

.editors-center {
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
    padding: 0 8px;
}

.query-section {
    background: #fff;
    border-radius: 8px;
    padding: 16px;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.06);
}

.query-tabs {
    max-width: 640px;
}

.query-tabs >>> .el-tabs__header {
    margin-bottom: 12px;
}
</style>