<template>
    <div class="panel">
        <div class="title">修改部门</div>
        <div class="div-margin">
            <span class="sign-text">修改部门：</span>
            <el-select filterable remote v-model="department_name"
                       style="width: 200px;"
                       placeholder="请输入要修改的部门"
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
        <div class="div-margin">
            <span class="sign-text">部门编号：</span>
            <el-input :value="now_department.department_id" style="width: 200px;" disabled></el-input>
        </div>
        <div class="div-margin">
            <span class="sign-text">部门名称：</span>
            <el-input
                v-model="new_department_name"
                :disabled="ifDisable"
                style="width: 200px;"></el-input>
        </div>
        <el-button :icon="Edit" round plain color="#76B6EA" size="large"
                   style="width: 150px;height: 30px;margin-top: 20px;margin-left: 65px;margin-bottom: 25px" @click="editDepartment"
        >修改部门
        </el-button>

    </div>
</template>

<script lang="ts" setup>
import {reactive, ref} from 'vue'
import {Edit} from '@element-plus/icons-vue'
import {ElMessage} from "element-plus";


const props = defineProps(['ori_department','department_map'])

/* 定义内部用变量 */
interface ListItem {
    value: string
    label: string
}

const ifDisable = ref(true)
const department_name = ref("")     // 要修改的部门名称
const new_department_name = ref("") // 修改的部门名称
let now_department = {
    department_id: "",      // 机构id
    top_department: "",      // 所属机构
    sub_department: [],      // 下属机构
    department_name: "",     // 机构名称
    leader: "",             // 主管id（1名）
    vice_leader: [],        // 副主管id（多名）
    members: []              // 办事人员
}     // 要编辑的部门
const loading = ref(false)          // 查找部门时，是否显示正在搜索
const options = ref<ListItem[]>([]) // 查找到的可选部门
const list = ref<ListItem[]>([])    // 搜索部门列表


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
    /* 给各个变量赋值 */
    ifDisable.value = false
    new_department_name.value = now_department.department_name
}


const editDepartment = () => {
    if(now_department.department_id!==props.ori_department){
        let top_department_id = now_department.top_department
        let top_department = props.department_map.get(top_department_id)
        for(let i=0;i<top_department.sub_department.length;i++)
            if(props.department_map.get(top_department.sub_department[i]).department_name === new_department_name.value){
                ElMessage.error("部门名称："+new_department_name.value+"在同级组织中已经出现，请更换名称后重新修改！")
                return
            }
        now_department.department_name = new_department_name.value
        props.department_map.delete(now_department.department_id)
        props.department_map.set(now_department.department_id, now_department)
        ElMessage.success("修改成功！")
    }
    else
        ElMessage.error("错误：顶层组织名禁止修改！")


    /* 清空变量 */
    ifDisable.value = true
    department_name.value = ""
    new_department_name.value = ""
    now_department = {
        department_id: "",      // 机构id
        top_department: "",      // 所属机构
        sub_department: [],      // 下属机构
        department_name: "",     // 机构名称
        leader: "",             // 主管id（1名）
        vice_leader: [],        // 副主管id（多名）
        members: []              // 办事人员
    }
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