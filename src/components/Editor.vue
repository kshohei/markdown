<template>
<div class="editor">
    <h1> エディター画面 </h1>
    <span>{{ user.displayName }}</span>
    <button @click="logout"> ログアウト </button>
    <div class="editorWrapper">
        <div class="momeListWrapper">
            <div class="memoList" v-for="(memo,index) in memos" @click="selectMemo(index)" :data-selected="index == selectedIndex">
                <p class="memoTitle">{{displayTitle(memo.markdown)}}</p>
            </div>
            <button class="addMemoBtn" @click="addMemo">メモの追加</button>
            <button class="deleteMemoBtn" v-if="memos.length>1" @click="deleteMemo">選択中のメモの削除</button>
            <button class="deletaAllMemosBtn" v-if="memos.length>1" @click="deleteAllMemos">全てのメモを削除</button>
            <button class="savaMemoBtn" @click="saveMemos">メモの保存</button>
        </div>
        <textarea class="markdown" v-model="memos[selectedIndex].markdown"></textarea>
        <div class="preview" v-html="preview()"></div>
    </div>
</div>
</template>

<script>
import marked from 'marked';

var len = 1;
export default {
    name: 'editor',
    props: ['user'],
    data() {
        return {
            memos: [{
                markdown: '1'
            }],
            selectedIndex: 0
        }
    },
    created() {
        firebase
            .database()
            .ref('memos/' + this.user.uid)
            .once('value')
            .then(result => {
                if (result.val()) {
                    this.memos = result.val();
                }
            })
    },
    methods: {
        logout: function () {
            firebase.auth().signOut();
        },
        addMemo: function () {
            len++;
            this.memos.push({
                markdown: "" + len + "",
            })
        },
        selectMemo: function (index) {
            this.selectedIndex = index;
        },
        deleteMemo: function () {
            this.memos.splice(this.selectedIndex, 1);
            if (this.selectedIndex > 0) {
                this.selectedIndex--;
            }
        },
        saveMemos: function () {
            firebase
                .database()
                .ref('memos/' + this.user.uid)
                .set(this.memos);
        },
        deleteAllMemos: function (){
            this.memos.push({
                markdown: '1'
            })
            len = 1
            this.memos.splice(0,this.memos.length-1);

            //  firebase
            //     .database()
            //     .ref('memos/' + this.user.uid)
            //     .set(null);
        },
        preview: function () {
            return marked(this.memos[this.selectedIndex].markdown);
        },
        displayTitle: function (text) {
            return text.split(/\n/)[0];
        },
    }
}
</script>

<style lang="scss" scoped>
.editorWrapper {
    display: flex;
}

.memoListWrapper {
    width: 19%;
    float: left;
    border-top: 1px solid #000;
}

.memoList {
    padding: 10px;
    box-sizing: border-box;
    text-align: left;
    border-bottom: 1px solid #000;
    &:nth-child(even) {
        background-color: #ccc;
    }
    &[data-selected="true"] {
        background-color: #ccf;
    }
}

.memoTitle {
    height: 1.5em;
    margin: 0;
    white-space: nowrap;
    overflow: hidden;
}

.addMemoBtn {
    margin-top: 20px;
}

.deleteMemoBtn {
    margin: 10px;
}

.deletaAllMemosBtn {
    margin:10px;
}

.markdown {
    float: left;
    width: 40%;
    height: 500px;
}

.preview {
    float: left;
    width: 40%;
    text-align: left;
}
</style>
