<template>
    <div id="app" @contextmenu="rightBtnMenuFn($event)" @drop="updateFile">
        <div class="container">
            <h2>简易的文件管理系统</h2>
            <div class="box">
                <div class="positionUrl" id="positionUrl">
                    当前位置：<span>{{fileData.name}}</span> <span v-for="item in url"> > {{item}}</span>
                </div>
                <ul class="folder" v-if="fileList.child.length > 0">
                    <li v-for="item in fileList.child" @click="selectThis">
                        <img v-if="item.type == 'file'"  src="./assets/file.png" alt="file">
                        <img v-if="item.type == 'folder'" @dblclick="openFolder" src="./assets/folder.png" alt="folder">
                        <h4 class="names" @dblclick="resetName" title="item.name">{{item.name}}</h4>
                    </li>
                </ul>
            </div>
        </div>
        <div id="rightBtnMenu" class="rightBtnMenu" v-show="rightBtnMenuIsShow">
            <ul>
                <li @click="rightBtnEvent" v-for="(item,n) in rightBtnMenu"><img :src="'/static/rightBtnMenu'+n+'.png'" :alt="item">{{item}}</li>
            </ul>
        </div>
        <div class="editInfo" id="editInfo" v-show="infoShow">
            <input type="text" placeholder="请输入文字" class="form-control" id="infoValue">
            <button class="btn btn-default" id="sure">确定</button>
            <button class="btn btn-default" id="cancel">取消</button>
        </div>
        <div id="wrap" v-if="infoShow"></div>
    </div>
</template>

<script>
import  '../node_modules/bootstrap/dist/css/bootstrap.min.css'

export default {
    data:function(){
        return {
            fileList:{
                name:"",
                type:"",
                child:[]
            },
            fileData:{
                name:"home",
                type:"",
                child:[
                    
                ]
            },
            url:[],
            urlSort:[],
            rightBtnMenu:["复制","粘贴","刷新","新建文件夹","删除"],
            rightBtnMenuIsShow:false,
            infoShow:false,
            obj:null,
            clipboard:null,
            infoValue:""
        }
    },
    methods:{
    //右键菜单系列
        //显示右键菜单
        rightBtnMenuFn:function(event){
            var event = event || window.e;
            var rightBtn = document.getElementById("rightBtnMenu");
                this.obj = event.target;
            var _this = this
            var noClick = function(event){
                    if (event.target.id != "rightBtnMenu"){

                       _this.rightBtnMenuIsShow = false;
                       document.removeEventListener("click",noClick,false);
                    }
                }

            event.preventDefault();
            rightBtn.style="position:fixed;left:"+event.clientX+"px;top:"+event.clientY+"px";
            this.rightBtnMenuIsShow = true;
            return false;
            
            document.addEventListener("click",noClick,false) 
        },
        rightBtnEvent:function(event){
            var event = event || window.e;
            var text = event.target.innerText;
            
            switch (text){
                case "刷新":
                    this.refresh();
                    break;
                case "复制":
                    this.copy();
                    break;
                case "粘贴":
                    this.paste();
                    break;
                case "新建文件夹":
                    this.addFolder();
                    break;
                case "删除":
                    this.deleted();
                    break;
                default:;
            }
            this.rightBtnMenuIsShow = false;
        },
        refresh:function(){
            var url = window.location.href;
            window.location.href = url;
        },
        addFolder:function(){
            var obj = {
                name:"新建文件夹",
                type:"folder",
                child:[]
            }

            this.createNewList(obj)
        },
        copy:function(){

            if(this.obj.nodeName == "IMG"){
                var obj =this.obj.parentNode;
                var index = this.listIndex(obj);
                var currentObj = this.getfileList().child[index];

                function clone(obj){
                    var newObj = new Object();
                    if(typeof obj !="object" || obj == null){
                        return obj
                    }
                    if (obj instanceof Array){
                        newObj = new Array();
                    }
                    
                    for (var i in obj){
                        console.log(clone(obj[i]))
                        newObj[i] = clone(obj[i]);
                    }
                    return newObj
                }
                
                this.clipboard = clone(currentObj)
                
            }
        },
        paste:function(){
            if (this.clipboard){
                this.createNewList(this.clipboard);
            }else {
                console.log("剪贴板为空");
            }
        },
        downFile:function(){
            console.log("下载选中的文件");
        },
        deleted:function(){
            if(this.obj.nodeName == "IMG"){
                var obj =this.obj.parentNode;  
                var index = this.listIndex(obj);

                if(confirm("是否真的要删除，不可恢复")){
                    this.getfileList().child.splice(index,1);
                    this.saveData();
                }

            }
        },
    //文件或者文件夹操作
        //创建新文件或者文件夹
        createNewList:function(newList){

            if(typeof newList == "object" && newList != null){
                var currentObj = this.getfileList();
                currentObj.child.push(newList);
                this.saveData();
            }
        },
        
        //打开文件夹
        openFolder:function(event){
            var _this =this;
            var index = this.listIndex(event.target.parentNode);
            
            this.urlSort.push(index);

            setTimeout(function(){
                _this.position();
            },10)
        },
        //重命名
        resetName:function(event){
            
            var event = event || window.e;
            var _this = this;
            this.infoShow = true;

            var setFolderName = function(name){
                var index = _this.listIndex(event.target.parentNode);
                
                _this.getfileList().child[index].name = name;
                _this.saveData();
            }
            this.getInfo(setFolderName);

            setTimeout(function(){
                document.getElementById("infoValue").focus();
            },10)
        },
        selectThis:function(){

        },
        //弹窗获得修改信息
        getInfo:function(doshomthing){
            var _this =this;
            document.getElementById("sure").onclick = function(){
                _this.infoShow = false;
                var infoVal = document.getElementById("infoValue").value;
                if (infoVal != ""){
                    doshomthing(infoVal);
                }else {
                   alert("输入内容不能为空");
                }
            }
            document.getElementById("cancel").onclick =function(){
                _this.infoShow = false;
                return ;
            }   

        },
    //面包屑
        position:function(){

            var position = document.getElementById("positionUrl"),
                positionList = position.getElementsByTagName("span");

            var _this = this;

            for (var i = positionList.length - 1; i >= 0; i--) {
                
                positionList[i].addEventListener("click",function(){

                    var index = _this.listIndex(this);
                    var count =  _this.urlSort.length-index;
                    _this.urlSort.splice(index,count);
                },false)
            }

        },
    //拖放文件
        updateFile:function(event){
            var event = event||window.e;
            var file = event.dataTransfer.files;
            console.log(file)

            for (var i = 0; i < file.length; i++) {
                var obj = {
                    name:file[i].name,
                    type:"file"
                }
                this.createNewList(obj);
            }
        },
        //获取索引
        listIndex:function(obj){
            
            for (var i = obj.parentNode.children.length - 1; i >= 0; i--) {
                 if (obj.parentNode.children[i] == obj){
                    return i;
                 }
             } 
        },
        //取得当前目录
        getfileList:function(){
            var obj = this.fileData;
            
            if (this.urlSort.length>0){

                for (var i = 0; i < this.urlSort.length; i++) {

                    var index = this.urlSort[i];

                    obj = obj.child[index];
                }
            }
            return obj;
        },
        //保存数据到本地
        saveData:function(){
            localStorage.fileData = JSON.stringify(this.fileData);
        }
    },
    mounted:function(){
        if(localStorage.fileData){

            this.fileData = JSON.parse(localStorage.getItem("fileData"));
        };
        this.fileList = this.fileData;
        this.position();
        var moren = function(event){
            var event = event || window.e;
            event.preventDefault();
        }
        document.getElementById("app").ondragleave = moren;
        document.getElementById("app").ondrop = moren;
        document.getElementById("app").ondragenter = moren;
        document.getElementById("app").ondragover = moren;
    },
    watch:{
        //监听面包屑的变化
        urlSort:function(){
            var obj = this.fileData;
            this.url = [];

            if (this.urlSort.length>0){

                for (var i = 0; i < this.urlSort.length; i++) {

                    var index = this.urlSort[i];

                    obj = obj.child[index];

                    this.url.push(obj.name);
                }
            }
            this.fileList = obj;
        }
    }
}
/*
困难 1、复制的文件夹下新建文件等操作同时会影响原文件夹。是因为对象浅复制造成的。决解办法 定义一个clone()函数深层复制
困难 2、面包屑的设置。解决办法。创建一个数组，保存打开了那一个文件夹的索引，一层一层记录下去。

进步：学会了html5的文件拖拽功能，对象的深浅复制。右键菜单制作。
*/
</script>

<style>
    html,body {
        width: 100%;
        height: 100%;
    }
    #app {
      font-family: 'Avenir', Helvetica, Arial, sans-serif;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
      color: #2c3e50;
    }
    ul,li {
        margin:0;
        padding:0;
    }
    li {
        list-style:none;
    }
    .box {
        border:1px solid #ccc;
        min-height: 400px;
    }
    .positionUrl {
        line-height: 40px;
        font-size:16px;
        padding:0 20px;
    }
    .positionUrl span{
        color:#49a5dc;
        cursor: pointer;
    }
    .folder {
        text-align: left;
        padding:20px;
    }
    .folder li {
        display: inline-block;
        max-width: 79px;
        margin: 2px 16px;
        padding:1px 8px;
        text-align: center;  
        border-radius: 2px; 
    }
    .folder li:hover {
        opacity: 0.8;
        background: #e9f8fd; 
        border:1px solid #9db6f9; 
        padding:0 7px;
    }
    .folder li img {
        width: 60px;
        height: auto;
        cursor: pointer;
    }
    .names { 
        width: 100%;
        font-size:12px;
        text-align: center;
        overflow: hidden;
        white-space: nowrap;
        text-overflow: ellipsis;
    }  
    .rightBtnMenu {
        width: 150px;
        min-height: 150px;
        border:1px solid #ccc;
        box-shadow: 0 0 1px 1px #ccc; 
        z-index: 999;
        background: #fff;
        border-radius: 3px;
    }
    .rightBtnMenu {
        padding:10px;
    }
    .rightBtnMenu li {
        line-height: 35px;
        cursor: pointer;
    }
    .rightBtnMenu li:hover {
        font-weight: bold;
    }
    .rightBtnMenu img {
        padding-right:5px;
        border-right:1px solid #ccc;
        margin-right: 5px;
    }
    #wrap {
        width: 100%;
        height: 100%;
        position: fixed;
        left: 0;
        top:0;
        background: #000;
        opacity: 0.6;
        z-index: 999;
    }
    .editInfo {
        width: 200px;
        height: auto;
        position: fixed;
        left: 50%;
        margin-left: -75px;
        top: 50%;
        z-index: 1000;
        background: #fff;
        border-radius: 5px;
        border:1px solid #ccc; 
        padding:10px;
    }
    .editInfo input {
        margin-bottom: 10px;
    }
</style>
