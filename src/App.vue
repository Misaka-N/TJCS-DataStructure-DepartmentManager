<template>
    <section v-if="showIndex === 0">
        <div class="logo"><img class="tree" src="./assets/icon/logo.svg"/></div>
        <div class="caption"><span style="letter-spacing: 0;"></span>高校职工管理系统</div>
        <ul class="options">
            <el-input v-model="name"  placeholder="请输入组织机构名" clearable
                      style="width: 400px;
                             height: 50px;
                             margin-left: 100px;
                             font-size: 20px"/>
            <li @click="changePage()">开始</li>
        </ul>
    </section>
    <section v-else-if="showIndex === 1">
        <div class="title">{{ name }}</div>
        <Manage :name="name" :origin_department_id="origin_department_id"></Manage>
    </section>

</template>

<script setup>
import {ref} from 'vue'
import {ElMessage} from 'element-plus'
import Manage from "@/components/Manage.vue";

const showIndex = ref(0)        // 当前要显示哪个网页
const name = ref("")            // 组织机构名
const origin_department_id = ref('')    //最顶层的机构编号

const checkTime = function (time) {
    if (time < 10) {
        time = "0" + time;
    }
    return time;
}

const changePage = () => {
    if(name.value === ""){
        ElMessage.error("组织机构名不能为空！")
        return
    }

    showIndex.value = 1

    let nowData = new Date()
    let year = '00',
        month = checkTime(nowData.getMonth() + 1),
        day = checkTime(nowData.getDate()),
        hour = checkTime(nowData.getHours()),
        min = checkTime(nowData.getMinutes()),
        sec = checkTime(nowData.getSeconds())
    origin_department_id.value = year + month + day + hour + min + sec
}


</script>

<style scoped>
.logo {
    width: 200px;
    height: 200px;
    background-color: rgba(255, 255, 255, 0.6);
    border-radius: 1000px;
    position: absolute;
    top: 30%;
    left: 50%;
    margin-left: -100px;
    margin-top: -100px;
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
}

.logo img {
    width: 60%;
    height: 60%;
    padding-left: 40px;
    padding-top: 40px;
}

.caption {
    height: 60px;
    width: 600px;
    font-family: STZhongsong;
    /*font-weight: bold;*/
    font-size: 56px;
    font-weight: bold;
    letter-spacing: 5px;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
    color: white;
    text-align: center;
    position: absolute;
    top: 45%;
    left: 50%;
    margin-left: -300px;
}

.title {
    height: 60px;
    width: 600px;
    font-family: STZhongsong;
    /*font-weight: bold;*/
    font-size: 48px;
    font-weight: bold;
    letter-spacing: 3px;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
    color: white;
    text-align: center;
    position: absolute;
    top: 3%;
    left: 50%;
    margin-left: -300px;
}

.options {
    display: block;
    width: 600px;
    height: 300px;
    list-style-type: none;
    position: absolute;
    top: 60%;
    left: 50%;
    margin-left: -300px;
}

.options li {
    display: block;
    width: 300px;
    height: 75px;
    background-color: rgba(255, 255, 255, 0.4);
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
    border-radius: 1000px;
    font-family: STZhongsong;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
    color: white;
    font-size: 36px;
    margin: auto;
    margin-top: 50px;
    text-align: center;
    padding-top: 17px;
    transition: box-shadow 0.5s ease;
    letter-spacing: 10px;
}

.options li:hover {
    box-shadow: 0 0 30px rgba(0, 0, 0, 0.5);
}

</style>