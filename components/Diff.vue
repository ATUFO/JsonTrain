<template>
    <div class="diff-page">
        <header class="app-header">
            <span class="app-title">JSON TRAIN</span>
            <nav class="app-nav">
                <nuxt-link class="nav-link" to="/">Train</nuxt-link>
                <nuxt-link class="nav-link" to="/diff">Diff</nuxt-link>
                <a class="nav-link nav-link--external" target="_blank" rel="noopener" href="https://github.com/ATUFO/JsonTrain">GitHub</a>
            </nav>
        </header>
        <main class="diff-main">
            <div class="diff-editors">
                <div class="editor-panel">
                    <div class="editor-panel__label">JSON A（旧）</div>
                    <div class="editor-panel__body">
                        <vue-json-editor
                            class="json-editor"
                            v-model="textb"
                            :showBtns="false"
                            :mode="'code'"
                            @json-change="onJsonChange"
                            @json-save="onJsonSave"
                            @has-error="onError"
                        />
                    </div>
                </div>
                <div class="editor-panel">
                    <div class="editor-panel__label">JSON B（新）</div>
                    <div class="editor-panel__body">
                        <vue-json-editor
                            class="json-editor"
                            v-model="texta"
                            :showBtns="false"
                            :mode="'code'"
                            @json-change="onJsonChange"
                            @json-save="onJsonSave"
                            @has-error="onError"
                        />
                    </div>
                </div>
            </div>
            <div class="diff-toolbar">
                <el-button type="primary" @click="handleXDiff">X Diff</el-button>
                <el-checkbox v-model="sort">排序</el-checkbox>
                <el-checkbox v-model="full">全部</el-checkbox>
            </div>
            <div class="diff-result-wrap">
                <div class="diff-result-label">Diff 结果</div>
                <pre v-highlightjs="xDiffResult" class="diff-result"><code class="diff"></code></pre>
            </div>
        </main>
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
<style scoped>
.diff-page {
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
    color: rgba(255, 255, 255, 0.9);
    padding: 6px 12px;
    border-radius: 6px;
    font-size: 14px;
    text-decoration: none;
}

.nav-link:hover,
.nav-link.nuxt-link-active {
    color: #fff;
    background: rgba(255, 255, 255, 0.15);
}

.nav-link--external {
    margin-left: 8px;
}

.diff-main {
    padding: 20px;
    max-width: 100%;
    box-sizing: border-box;
}

.diff-editors {
    display: flex;
    gap: 12px;
    margin-bottom: 16px;
    height: 38vh;
    min-height: 200px;
}

.diff-editors .editor-panel {
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

.diff-toolbar {
    display: flex;
    align-items: center;
    gap: 16px;
    margin-bottom: 16px;
    padding: 12px 0;
}

.diff-result-wrap {
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.06);
    overflow: hidden;
}

.diff-result-label {
    padding: 10px 14px;
    font-size: 13px;
    font-weight: 600;
    color: #303133;
    background: #fafafa;
    border-bottom: 1px solid #ebeef5;
}

.diff-result {
    margin: 0;
    padding: 16px;
    min-height: 40vh;
    max-height: 50vh;
    overflow: auto;
    font-size: 13px;
    line-height: 1.5;
}

.diff-result code {
    display: block;
    min-height: 100%;
}
</style>
