<template>
<div class="main">
  <div id="title" class="center">LL(1)分析表的求取</div>
  <div v-if="isIn" class="content">
      <div class="center">
        <div id="input">
          <el-input type="textarea"
                    :autosize="{ minRows: 14, maxRows: 17}"
                    resize="none"
                    v-model="input"
                    placeholder="请输入文法，如S-AB|C">
          </el-input>
        </div>
      </div>
  </div>
  <div v-else class="content">
    <el-table
      :data="tableData">
      <div v-for="item in right">
      <el-table-column
        :prop="prop"
        :label="item"
      >
        {{grammars[i]}}
      </el-table-column>
      </div>
    </el-table>
  </div>
  <el-button type="primary" class="center" @click="turnIn">{{textIn}}</el-button>
</div>
</template>

<script>
    export default {
        name: "ll1",
        data(){
          return {
            input:'',
            isIn:true,
            textIn:"查看结果",

            grammars:[],
            left:[],
            right:[],
            first:[],
            follow:[],
            tableData:[],
          }
        },

      mounted(){
          this.input='E->Te\n' +
        'e->+Te|ε\n' +
        'T->Ft\n' +
        't->*Ft|ε\n' +
        'F->(E)|i\n';
          this.turnIn();
      },
//ε
        methods:{

          turnIn:function(){
            if(this.isIn){
              this.isIn=false;
              this.getStr();
              this.setFirst();
              this.setFollow();
              this.setTable();
              this.textIn="重新输入";
            }else{
              this.isIn=true;
              this.grammars=[];
              this.left=[];
              this.right=[];
              this.follow=[];
              this.first=[];
              this.textIn="查看结果";
            }
          },

          getStr:function(){
            let ss = this.input.split('\n');
            let right=[];
            for(let i = 0;i<ss.length;i++){
              if(ss[i].length>0){
                let g = ss[i].split("->");
                let r = g[1].split('|');
                this.left.push(g[0]);
                right.push(g[1]);
                let grammar = {
                  left:g[0],
                  right:r,
                };
                this.grammars.push(grammar);
              }
            }
            //剔除重复及|
            right = Array.from(new Set(this.right.join('').split('')));
            if(right.indexOf('|')!==-1) {
              right.splice(this.right.indexOf('|'), 1);
            }
            //剔除非终结符,初始化first和follow
            for(let i=0;i<this.left.length;i++) {
              this.first[i] = [];
              this.follow[i] = [];
              if (right.indexOf(this.left[i]) !== -1) {
                right.splice(this.right.indexOf(this.left[i]), 1);
              }
            }

            right.unshift("G","First","Follow");
            // for(let i=0;i<right.length;i++){
            //   let obj={};
            //
            // }
          },

          getFirst:function(index){
            let first = [], aim = 0 ,len = -1;
            for(let j = 0;j<this.grammars[index].right.length;j++) {
              let c = this.grammars[index].right[j][aim];
              let i = this.left.indexOf(c);
              if (i === -1) {//终结符
                first.push(c);
              } else if (this.first[i].length===0) {//非终结且未first
                first = first.concat(this.getFirst(i));
              }
              else {//未终结且已first
                first = first.concat(this.first[i]);
              }
              let blank = first.lastIndexOf('ε');
              if(blank>len&&this.grammars[index].right[j].length>aim+1){//存在空且存在下一个
                first.splice(blank,1);
                aim++;
                j--;
              }else{
                aim = 0;
              }
              len = first.length;
            }
            this.first[index] = first;
            return first;
          },

          setFirst:function () {
            for(let i=0;i<this.grammars.length;i++){
              if(this.first[i].length===0)
                this.getFirst(i);
              this.first[i] = Array.from(new Set(this.first[i]));
              console.log(this.grammars[i].left+'->'+this.first[i]);
            }
          },

          getFollow:function(index) {
            let follow = [],aim = 0,len = -1;
            // console.log(this.left[index]);
            for (let i = 0; i < this.grammars.length; i++) {
              if (i === index) continue;
              for (let j = 0; j < this.grammars[i].right.length; j++) {
                let ri = this.grammars[i].right[j].indexOf(this.left[index]) + aim;
                if (ri === -1) continue;
                if(ri > this.grammars[i].right[j].length - 1){
                  follow = follow.concat(this.follow[i]);
                }else if (ri === this.grammars[i].right[j].length - 1) {//为最末位
                  if (this.follow[i].length === 0||index===0) {//未follow
                    follow = follow.concat(this.getFollow(i));
                  } else {//已follow
                    follow = follow.concat(this.follow[i]);
                  }
                } else {//不为最末
                  let nextc = this.grammars[i].right[j][ri + 1];
                  let nexti = this.left.indexOf(nextc);
                  if (nexti === -1) {//下一个为终结
                    follow.push(nextc);
                  } else {//下一个非终结
                    follow = follow.concat(this.first[nexti]);
                  }
                }

                // console.log(follow);

                let blank = follow.lastIndexOf('ε');
                if (blank > len) {//存在空
                  follow.splice(blank, 1);
                  aim++;
                  j--;
                } else {
                  aim = 0;
                }
                len = follow.length;
              }
            }
            this.follow[index] = this.follow[index].concat(follow);
            return follow;
          },

          setFollow:function () {
            this.follow[0]=['#'];
            for(let i=0;i<this.grammars.length;i++){
              if(this.follow[i].length===0||i===0){
                this.getFollow(i);
              }
              this.follow[i] = Array.from(new Set(this.follow[i]));
              console.log(this.grammars[i].left+'->'+this.follow[i]);
            }

          },

          setTable:function () {
            for(let i = 0;i<this.left.length;i++){
              let tmp = [this.left[i],this.first[i],this.follow[i]];
              for(let j = 3;j<this.right.length;j++){
                tmp[j]="";
              }
              for(let j = 0;j<this.first[i].length;j++){
                if(this.first[j]==='ε'){
                  for(let k = 3; k < this.right.length; k++){
                    if(this.follow[i].indexOf(this.right[k])!==-1){
                      tmp[k].concat(this.left[i]+"->ε\n");
                    }
                  }
                }else {
                  for (let k = 0; k < this.grammars[i].right.length; k++) {
                    if (this.grammars[i].right[k].indexOf(this.first[j]) !== -1) {
                      tmp[k].concat(this.left[i] + "->" + this.grammars[i].right[k] + "\n");
                    }
                  }
                }
              }
              let obj={};
              for(let j = 0;j<this.right.length;j++){
                console.log(this.right[j]);
                Object.defineProperty(obj,this.right[j],{
                  value:tmp[j],
                  writable:true,
                });
                console.log(this.right[j]);
              }
              this.tableData.push(obj);
            }
            console.log(this.tableData);
          },

        }
    }
</script>

<style>
  .el-textarea__inner{
    font-size: 1.2rem;
  }
</style>

<style scoped>
  .main{
    width: 100%;
    height: 100%;
    text-align: center;
  }
  #title{
    font-size: 1.5rem;
    margin: 30px;
  }
  .content{
    width: 100%;
    height:70%;
  }
  .center{
    margin: 0 auto;
    margin-top: 20px;
    display: block;
  }

  .input{
    width: 25%;
    margin:0 auto;
  }


</style>
