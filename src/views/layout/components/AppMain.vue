<template>
  <section class="app-main">
     <div style="height:100%;">
        <div v-loading="loading" element-loading-text="数据加载中..." element-loading-background="rgba(0, 0, 0, 0.8)" style="height: 100%;">
            <!-- <v-head ref="head"></v-head> -->
            <div style="padding-top: 0px" ></div>
            <router-view v-if='routerShow' class="main"  @addGroup='change'></router-view>
        </div>
        <add-group @close="addGroups" v-if="addGroupShow" @success='addSuccess' :show='addGroupShow'></add-group>
    </div>
  </section>
</template>
<script>
    // import headers from './headers'
    import addGroup from "@/components/addGroup"
    import { getGroupList,getEncryptType } from "@/api/api"
    import '@/assets/css/layout.css'
    import '@/assets/css/public.css'
    import url from '@/api/url'
    import router from '@/router'
    import constant from '@/util/constant'
    import {message} from '@/util/util'

    export default {
        name: 'AppMain',
        components:{
            // 'v-head': headers,
            "add-group": addGroup
        },
        data: function () {
            return {
                loading: false,
                addGroupShow: false,
                groupList: [],
                groupId: null,
                routerShow: false,
            }
        },
        mounted: function () {
          
            this.$nextTick(function () {
                this.GetgroupList();
            })
        },
        methods: {
            change: function () {
                this.$refs.head.getGroupData();
            },
            GetgroupList: function(){
                let data = {};
                getGroupList(data).then(res => {
                    if(res.data.code === 0){
                        this.routerShow = true
                        if(res.data.data.length){
                            this.groupList = res.data.data
                            if(!localStorage.getItem("groupId")){
                                this.groupId = res.data.data[0].groupId;
                                localStorage.setItem("groupId",this.groupId);
                            }
                            localStorage.setItem("groupList",JSON.stringify(this.groupList))
                            // this.getEncrypt(); 无效请求404
                            this.change();
                            if(this.$route.query.pkHash){
                                router.push({
                                    name: this.$route.query.path,
                                    query: {
                                        pkHash: this.$route.query.pkHash
                                    }
                                })
                            }else if(this.$route.query.blockHash){
                                router.push({
                                    name: this.$route.query.path,
                                    query: {
                                        blockHash: this.$route.query.blockHash
                                    }
                                })
                            }else{
                                router.push({
                                    name: this.$route.query.path,
                                })
                            }
                            
                        }else{
                            router.push({
                                name: "groupConfig"
                            })
                        }
                    }else{
                        message(res.data.message,'error')
                    }
                }).catch(err => {
                    message(constant.ERROR,'error');
                })
            },
            getEncrypt: function(){
                getEncryptType(localStorage.getItem("groupId")).then(res => {
                    if(res.data.code === 0){
                        localStorage.setItem("encryptionId",res.data.data)
                    }else{
                        message(errorcode[res.data.code].cn,'error')
                    }
                }).catch(err => {
                    message(constant.ERROR,'error');
                })
            },
        }
    }
</script>


<style scoped>
.app-main {
  /*50 = navbar  */
  min-height: calc(100vh - 50px);
  position: relative;
  overflow: hidden;
}
</style>
