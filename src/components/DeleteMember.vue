<template>
    <div class="panel">
        <div class="title">删除人员</div>
        <div class="div-margin">
            <span class="sign-text">删除人员：</span>
            <el-select filterable remote v-model="member_name"
                       style="width: 200px;"
                       placeholder="请输入要删除的人员"
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
            <span class="sign-text">人员编号：</span>
            <el-input :value="now_member.member_id" style="width: 200px;" disabled></el-input>
        </div>
        <div class="div-margin">
            <span class="sign-text">所属部门：</span>
            <el-input
                :value="now_member.member_position.length === 0 ? '暂无' : department_map.get(now_member.member_position[0].department_id).department_name + '等'+ now_member.member_position.length + '个'"
                style="width: 200px;" disabled></el-input>
        </div>
        <el-button :icon="Delete" round plain color="#F7CAC9" size="large"
                   style="width: 150px;height: 30px;margin-top: 20px;margin-left: 65px;margin-bottom: 25px" @click="dialogVisible=true"
        >删除人员
        </el-button>

        <el-dialog
            v-model="dialogVisible"
            title="注意！"
            width="30%"
            :append-to-body="false"
            :z-index="2000"
        >
            <span>此操作将删除当前人员及其所有部门职位，确定继续？</span>
            <template #footer>
                <span class="dialog-footer">
                    <el-button @click="dialogVisible = false">取消</el-button>
                    <el-button type="danger" @click="deleteMember">
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


const props = defineProps(['department_map', 'member_map'])
const emit = defineEmits(['deleteMember'])

/* 定义内部用变量 */
interface ListItem {
    value: string
    label: string
}

const dialogVisible = ref(false)
const member_name = ref("")     // 要删除的人员名称
let now_member = {
    member_name: "",        // 员工的姓名
    member_id: "",          // 员工的id
    member_position: []
}     // 要删除的部门
const loading = ref(false)          // 查找人员时，是否显示正在搜索
const options = ref<ListItem[]>([]) // 查找到的可选人员
const list = ref<ListItem[]>([])    // 搜索人员列表


const remoteMethod = (query: string) => {
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


const loadInfo = () => {
    let i = member_name.value.length - 1
    for (; i >= 0; i--)
        if (member_name.value[i] === '(')
            break
    let department_id = member_name.value.substring(i + 1, member_name.value.length - 1)
    now_member = props.member_map.get(department_id)
}


const deleteMember = () => {
    dialogVisible.value = false
    member_name.value = ""

    for(let i = 0; i<now_member.member_position.length;i++){
        let department = props.department_map.get(now_member.member_position[i].department_id)
        if(now_member.member_position[i].position === '0')
            department.members.splice(department.members.indexOf(now_member.member_id),1)
        else if(now_member.member_position[i].position === '1')
            department.vice_leader.splice(department.vice_leader.indexOf(now_member.member_id),1)
        else
            department.leader = ''
    }

    props.member_map.delete(now_member.member_id)

    now_member = {
        member_name: "",        // 员工的姓名
        member_id: "",          // 员工的id
        member_position: []
    }
    emit("deleteMember", props.department_map, props.member_map)
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