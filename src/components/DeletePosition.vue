<template>
    <div class="panel">
        <div class="title">删除职位</div>
        <div class="div-margin">
            <span class="sign-text">人员姓名：</span>
            <el-select filterable remote v-model="member_name"
                       style="width: 200px;"
                       placeholder="请输入人员姓名"
                       :remote-method="remoteMethodName"
                       :loading="loading"
                       @change="loadMemberInfo">
                <el-option
                    v-for="item in options"
                    :key="item.label"
                    :label="item.value +'('+item.label+')' "
                    :value="item.value +'('+item.label+')' "
                />
            </el-select>

        </div>
        <div class="div-margin">
            <span class="sign-text">所属部门：</span>
            <el-select filterable remote v-model="department_name"
                       style="width: 200px;"
                       placeholder="请输入所属部门"
                       :remote-method="remoteMethodDepartment"
                       :loading="loading"
                       @change="loadDepartmentInfo">
                <el-option
                    v-for="item in options"
                    :key="item.label"
                    :label="item.value +'('+item.label+')' "
                    :value="item.value +'('+item.label+')' "
                />
            </el-select>
        </div>
        <div class="div-margin">
            <span class="sign-text">所属职位：</span>
            <el-select v-model="member_position" style="width: 200px" disabled>
                <el-option label="办事人员" value="0"/>
                <el-option label="副主管" value="1"/>
                <el-option label="主管" value="2"/>
            </el-select>
        </div>
        <el-button :icon="Delete" round plain color="#F7CAC9" size="large"
                   style="width: 150px;height: 30px;margin-top: 20px;margin-left: 65px" @click="deletePosition"
        >删除人员职位
        </el-button>
    </div>
</template>

<script lang="ts" setup>
import {reactive, ref} from 'vue'
import {Delete} from '@element-plus/icons-vue'
import {ElMessage} from "element-plus";


const props = defineProps(['department_map', 'member_map'])
const emit = defineEmits(['deletePosition'])

/* 定义内部用变量 */
interface ListItem {
    value: string
    label: string
}

const member_name = ref("")     // 要删除的人员名称
let now_department = {
    department_id: "",      // 机构id
    top_department: "",      // 所属机构
    sub_department: [],      // 下属机构
    department_name: "",     // 机构名称
    leader: "",             // 主管id（1名）
    vice_leader: [],        // 副主管id（多名）
    members: []              // 办事人员
}
const department_name = ref("") // 要删除的职位对应的部门名称
let now_member = {
    member_name: "",        // 员工的姓名
    member_id: "",          // 员工的id
    member_position: []
}     // 要删除的部门
const member_position_index = ref(-1)
const member_position = ref("0")
const loading = ref(false)          // 查找人员时，是否显示正在搜索
const options = ref<ListItem[]>([]) // 查找到的可选人员
const list = ref<ListItem[]>([])    // 搜索人员列表


const remoteMethodName = (query: string) => {
    list.value = []
    props.member_map.forEach((val, key) => {
        list.value.push({value: val.member_name, label: key})
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

const remoteMethodDepartment = (query: string) => {
    list.value = []
    for (let i = 0; i < now_member.member_position.length; i++)
        list.value.push({
            value: props.department_map.get(now_member.member_position[i].department_id).department_name,
            label: now_member.member_position[i].department_id
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

const loadMemberInfo = () => {
    let i = member_name.value.length - 1
    for (; i >= 0; i--)
        if (member_name.value[i] === '(')
            break
    let member_id = member_name.value.substring(i + 1, member_name.value.length - 1)
    now_member = props.member_map.get(member_id)
}

const loadDepartmentInfo = () => {
    let i = department_name.value.length - 1
    for (; i >= 0; i--)
        if (department_name.value[i] === '(')
            break
    let department_id = department_name.value.substring(i + 1, department_name.value.length - 1)
    now_department = props.department_map.get(department_id)

    console.log(department_id)
    console.log(now_department)

    for(let i=0;i<now_member.member_position.length;i++){
        if(now_member.member_position[i].department_id === department_id){
            member_position.value = now_member.member_position[i].position
            member_position_index.value = i
            break
        }
    }
}

const deletePosition = () => {
    member_name.value = ""
    department_name.value = ""
    member_position_index.value = -1

    if (now_member.member_position.length === 1) {
        ElMessage.error("删除失败：当前职位为员工唯一职位！")
        return
    }

    if (member_position.value === '0')
        now_department.members.splice(now_department.members.indexOf(now_member.member_id), 1)
    else if (member_position.value === '1')
        now_department.vice_leader.splice(now_department.vice_leader.indexOf(now_member.member_id), 1)
    else
        now_department.leader = ''

    for(let i=0;i<now_member.member_position.length;i++){
        if(now_member.member_position[i].department_id === now_department.department_id
        && now_member.member_position[i].position === member_position.value){
            now_member.member_position.splice(i,1)
            break
        }
    }

    member_position.value = '0'

    emit("deletePosition", props.department_map, props.member_map)
    ElMessage.success("删除成功！")
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