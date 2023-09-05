<template>
    <div class="panel">
        <div class="title">查找部门</div>
        <div class="div-margin">
            <span class="sign-text">查找部门：</span>
            <el-select filterable remote v-model="department_name"
                       style="width: 200px;"
                       placeholder="请输入要查找的部门"
                       :remote-method="remoteMethod"
                       :loading="loading"
                       @change="loadInfo">
                <el-option
                    v-for="item in options"
                    :key="item.label"
                    :label="item.value +'('+item.label+')' "
                    :value="item.value +'('+item.label+')' "
                />
            </el-select>

        </div>
        <el-button :icon="Search" round plain color="#F3C677" size="large"
                   style="width: 150px;height: 30px;margin-top: 20px;margin-left: 65px;margin-bottom: 25px" @click="searchDepartment"
        >查询部门
        </el-button>

    </div>
</template>

<script lang="ts" setup>
import {reactive, ref} from 'vue'
import {Search} from '@element-plus/icons-vue'
import {ElMessage} from "element-plus";


const props = defineProps(['department_map'])
const emit = defineEmits(['showResult'])

/* 定义内部用变量 */
interface ListItem {
    value: string
    label: string
}
const department_name = ref("")     // 要查找的部门名称
const loading = ref(false)          // 查找部门时，是否显示正在搜索
const options = ref<ListItem[]>([]) // 查找到的可选部门
const list = ref<ListItem[]>([])    // 搜索部门列表
let now_department = {
    department_id: "",      // 机构id
    top_department: "",      // 所属机构
    sub_department: [],      // 下属机构
    department_name: "",     // 机构名称
    leader: "",             // 主管id（1名）
    vice_leader: [],        // 副主管id（多名）
    members: []              // 办事人员
}     // 要编辑的部门

const remoteMethod = (query: string) => {
    list.value = []
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


const loadInfo = () => {
    let i = department_name.value.length - 1
    for (; i >= 0; i--)
        if (department_name.value[i] === '(')
            break
    let department_id = department_name.value.substring(i + 1, department_name.value.length - 1)
    now_department = {...props.department_map.get(department_id)}
}

const searchDepartment=()=>{
    if(department_name.value === '')
    {
        ElMessage.error('请输入部门名称！')
        return
    }
    let result_department = props.department_map.get(now_department.department_id)
    emit('showResult', true, result_department)
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