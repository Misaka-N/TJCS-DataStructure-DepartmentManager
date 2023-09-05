<template>
    <div class="panel">
        <div class="title">删除部门</div>
        <div class="div-margin">
            <span class="sign-text">删除部门：</span>
            <el-select filterable remote v-model="department_name"
                       style="width: 200px;"
                       placeholder="请输入要删除的部门"
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
            <span class="sign-text">所属部门：</span>
            <el-input
                :value="now_department.top_department === '' ? '暂无' : department_map.get(now_department.top_department).department_name + '('+ department_map.get(now_department.top_department).department_id + ')'"
                style="width: 200px;" disabled></el-input>
        </div>
        <div class="div-margin">
            <span class="sign-text">下属部门：</span>
            <el-input
                :value="now_department.sub_department.length === 0 ? '暂无' : (department_map.get(now_department.sub_department[0]).department_name  + ' 等'+now_department.sub_department.length+'个')"
                style="width: 200px;" disabled></el-input>
        </div>
        <el-button :icon="Delete" round plain color="#F7CAC9" size="large"
                   style="width: 150px;height: 30px;margin-top: 20px;margin-left: 65px;margin-bottom: 25px" @click="dialogVisible=true"
        >删除部门
        </el-button>

        <el-dialog
            v-model="dialogVisible"
            title="注意！"
            width="30%"
            :append-to-body="false"
            :z-index="2000"
        >
            <span>此操作将删除当前部门及其所有的下属部门，确定继续？</span>
            <template #footer>
                <span class="dialog-footer">
                    <el-button @click="dialogVisible = false">取消</el-button>
                    <el-button type="danger" @click="deleteDepartment">
                        确定
                    </el-button>
                </span>
            </template>
        </el-dialog>

    </div>
</template>

<script lang="ts" setup>
import {reactive, ref} from 'vue'
import {Delete} from '@element-plus/icons-vue'
import {ElMessage} from "element-plus";


const props = defineProps(['ori_department','department_map', 'member_map'])
const emit = defineEmits(['deleteDepartment'])

/* 定义内部用变量 */
interface ListItem {
    value: string
    label: string
}

const dialogVisible = ref(false)
const department_name = ref("")     // 要删除的部门名称
let now_department = {
    department_id: "",      // 机构id
    top_department: "",      // 所属机构
    sub_department: [],      // 下属机构
    department_name: "",     // 机构名称
    leader: "",             // 主管id（1名）
    vice_leader: [],        // 副主管id（多名）
    members: []              // 办事人员
}     // 要删除的部门
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
    now_department = props.department_map.get(department_id)
}


const myDelete = (thisDepartment) => {
    for (let i = 0; i < thisDepartment.sub_department.length; i++)
        myDelete (props.department_map.get(thisDepartment.sub_department[i]))
    if(thisDepartment.leader !== '')
    {
        let now_member_position = props.member_map.get(thisDepartment.leader).position
        for(let i=0;i<now_member_position.length;i++)
        {
            if(now_member_position[i].department_id === thisDepartment.department_id)
            {
                now_member_position.splice(i,1)
                if(now_member_position.length === 0)
                    props.member_map.delete(thisDepartment.leader)
                break;
            }
        }
    }
    if(thisDepartment.vice_leader.length !== 0)
    {
        for(let j=0;j<thisDepartment.vice_leader.length;j++)
        {
            let now_member_position = props.member_map.get(thisDepartment.vice_leader[j]).position
            for(let i=0;i<now_member_position.length;i++)
            {
                if(now_member_position[i].department_id === thisDepartment.department_id)
                {
                    now_member_position.splice(i,1)
                    if(now_member_position.length === 0)
                        props.member_map.delete(thisDepartment.vice_leader[j])
                    break;
                }
            }
        }
    }
    if(thisDepartment.members.length !== 0)
    {
        for(let j=0;j<thisDepartment.members.length;j++)
        {
            let now_member_position = props.member_map.get(thisDepartment.members[j]).member_position
            for(let i=0;i<now_member_position.length;i++)
            {
                if(now_member_position[i].department_id === thisDepartment.department_id)
                {
                    now_member_position.splice(i,1)
                    if(now_member_position.length === 0)
                        props.member_map.delete(thisDepartment.members[j])
                    break;
                }
            }
        }
    }
    props.department_map.delete(thisDepartment.department_id)
}


const deleteDepartment = () => {
    dialogVisible.value = false
    department_name.value = ""
    if(now_department.department_id === props.ori_department){
        ElMessage.error("错误：不能删除顶层部门！")
        return
    }
    let top_department_id = now_department.top_department
    let top_department = props.department_map.get(top_department_id)
    top_department.sub_department.splice(top_department.sub_department.indexOf(now_department.department_id), 1)
    myDelete(now_department)
    now_department = {
        department_id: "",      // 机构id
        top_department: "",      // 所属机构
        sub_department: [],      // 下属机构
        department_name: "",     // 机构名称
        leader: "",             // 主管id（1名）
        vice_leader: [],        // 副主管id（多名）
        members: []              // 办事人员
    }
    emit("deleteDepartment", props.department_map,props.member_map)
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