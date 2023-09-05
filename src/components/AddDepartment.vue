<template>
    <div class="panel">
        <div class="title">添加部门</div>
        <div class="div-margin">
            <span class="sign-text">部门名称：</span>
            <el-input v-model="department_name" placeholder="请输入新部门名称" style="width: 200px;" maxlength="15"
                      show-word-limit></el-input>
        </div>
        <div class="div-margin">
            <span class="sign-text">所属部门：</span>
            <el-select filterable remote v-model="top_department_name" :placeholder="'默认为'+props.name"
                       style="width: 200px;"
                       :remote-method="remoteMethod"
                       :loading="loading">
                <el-option
                    v-for="item in options"
                    :key="item.label"
                    :label="item.value +'('+item.label+')' "
                    :value="item.value +'('+item.label+')' "
                />
            </el-select>

        </div>
        <div class="div-margin">
            <span class="sign-text">部门编号：</span>
            <el-input v-model="department_id" style="width: 200px;" disabled></el-input>
        </div>
        <el-button :icon="Check" round plain color="#A2D9CE" size="large"
                   style="width: 150px;height: 30px;margin-top: 20px;margin-left: 65px;margin-bottom: 25px"
                   @click="checkInfo">添加部门
        </el-button>
    </div>
</template>

<script lang="ts" setup>
import {onMounted, reactive, ref} from 'vue'
import {Check} from '@element-plus/icons-vue'
import {ElMessage} from "element-plus";


const props = defineProps(['name', 'id', 'department_map'])
const emit = defineEmits(['addDepartment'])

/* 定义内部用变量 */
interface ListItem {
    value: string
    label: string
}

const department_name = ref("")     // 新部门名称
const department_id = ref("")     // 新部门编号
const top_department_name = ref("")     // 所属部门名称
const top_department_id = ref("")     // 所属部门编号
const loading = ref(false)          // 查找部门时，是否显示正在搜索
const options = ref<ListItem[]>([]) // 查找到的可选部门
const list = ref<ListItem[]>([])    // 搜索部门列表


onMounted(() => {
    getDepartmentId()
})

const getDepartmentId = ()=>{
    let nowData = new Date()
    let year = checkTime(nowData.getFullYear() % 2000),
        month = checkTime(nowData.getMonth() + 1),
        day = checkTime(nowData.getDate()),
        hour = checkTime(nowData.getHours()),
        min = checkTime(nowData.getMinutes()),
        sec = checkTime(nowData.getSeconds())
    department_id.value = year + month + day + hour + min + sec
}

const checkTime = function (time) {
    if (time < 10) {
        time = "0" + time;
    }
    return time;
}

const remoteMethod = (query: string) => {
    list.value=[]
    props.department_map.forEach((val, key) => {
        list.value.push({value: val.department_name, label: key})
    })
    if (query !== "") {
        loading.value = true
        setTimeout(() => {
            loading.value = false
            options.value = list.value.filter((item) => {
                return item.value.toLowerCase().indexOf(query.toLowerCase()) !== -1;
            })
        }, 200)
    } else {
        options.value = []
    }
}

const checkInfo = () => {
    if (department_name.value === "") {
        ElMessage.error("部门名不能为空！")
    }
    else{
        if (top_department_name.value === "")
            top_department_name.value = props.name + '(' + props.id + ')'

        let i = top_department_name.value.length - 1
        for (; i >= 0; i--)
            if (top_department_name.value[i] === '(')
                break
        top_department_id.value = top_department_name.value.substring(i + 1, top_department_name.value.length - 1)
        top_department_name.value = top_department_name.value.substring(0, i)

        let ifValid = true
        let top_department = props.department_map.get(top_department_id.value)
        for (let i = 0; i < top_department.sub_department.length; i++) {
            if (props.department_map.get(top_department.sub_department[i]).department_name === department_name.value) {
                ElMessage.error(department_name.value + "已经存在！（部门编号：" + props.department_map.get(top_department.sub_department[i]).department_id + "）")
                ifValid = false
                break
            }
        }
        if(ifValid){
            emit('addDepartment', department_name.value, department_id.value, top_department_name.value, top_department_id.value)
            ElMessage.success("添加成功！")
        }
    }
    top_department_id.value = ""
    top_department_name.value = ""
    department_name.value = ""
    getDepartmentId()
}
</script>

<style scoped>
.panel {
    width: 280px;
}

.title {
    text-align: center;
    color: #555555;
    letter-spacing: 3px;
    font-family: Msyh;
    font-size: 18px;
}

.sign-text {
    color: #555555;
    font-family: Msyh;
    font-size: 14px;
}

.div-margin {
    margin-top: 10px;
}
</style>
