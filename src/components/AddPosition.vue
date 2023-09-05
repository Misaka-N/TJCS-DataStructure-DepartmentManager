<template>
    <div class="panel">
        <div class="title">添加人员职位</div>
        <div class="div-margin">
            <span class="sign-text">人员姓名：</span>
            <el-select filterable remote v-model="member_name" placeholder="请输入人员姓名"
                       style="width: 200px;"
                       :remote-method="remoteMethodMember"
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
            <span class="sign-text">所属部门：</span>
            <el-select filterable remote v-model="top_department_name" placeholder="请输入对应部门"
                       style="width: 200px;"
                       :remote-method="remoteMethodDepartment"
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
            <span class="sign-text">人员职位：</span>
            <el-select v-model="member_position" placeholder="请选择人员职位" style="width: 200px">
                <el-option label="办事人员" value="0"/>
                <el-option label="副主管" value="1"/>
                <el-option label="主管" value="2"/>
            </el-select>
        </div>
        <el-button :icon="Check" round plain color="#A2D9CE" size="large"
                   style="width: 150px;height: 30px;margin-top: 20px;margin-left: 65px"
                   @click="checkInfo">添加人员职位
        </el-button>
    </div>
</template>

<script lang="ts" setup>
import {ref} from 'vue'
import {Check} from '@element-plus/icons-vue'
import {ElMessage} from "element-plus";


const props = defineProps(['name', 'id', 'member_map', 'department_map'])
const emit = defineEmits(['addPosition'])

/* 定义内部用变量 */
interface ListItem {
    value: string
    label: string
}

const member_name = ref("")     // 人员名称
const member_id = ref("")     // 人员id
const member_position = ref('0')  // 人员职位（默认为普通员工）
const top_department_name = ref("")     // 所属部门名称
const top_department_id = ref("")     // 所属部门编号
const loading = ref(false)          // 查找部门时，是否显示正在搜索
const options = ref<ListItem[]>([]) // 查找到的可选部门
const list = ref<ListItem[]>([])    // 搜索部门列表

const checkTime = function (time) {
    if (time < 10) {
        time = "0" + time;
    }
    return time;
}

const remoteMethodMember = (query: string) => {
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
    if (member_name.value === "") {
        ElMessage.error("人员姓名不能为空！")
    } else if (top_department_name.value === "") {
        ElMessage.error("所属部门不能为空！")
    } else {
        let i = top_department_name.value.length - 1
        for (; i >= 0; i--)
            if (top_department_name.value[i] === '(')
                break
        top_department_id.value = top_department_name.value.substring(i + 1, top_department_name.value.length - 1)
        top_department_name.value = top_department_name.value.substring(0, i)

        i = member_name.value.length - 1
        for (; i >= 0; i--)
            if (member_name.value[i] === '(')
                break
        member_id.value = member_name.value.substring(i + 1, member_name.value.length - 1)
        member_name.value = member_name.value.substring(1, i)

        let ifValid = true;

        if (props.department_map.get(top_department_id.value).members.indexOf(member_id.value) !== -1) {
            ElMessage.error("添加失败：要添加的员工已经存在于所属部门的办事人员列表中！")
            ifValid = false
        }
        if (props.department_map.get(top_department_id.value).vice_leader.indexOf(member_id.value) !== -1) {
            ElMessage.error("添加失败：要添加的员工已经存在于所属部门的副主管列表中！")
            ifValid = false
        }
        if (props.department_map.get(top_department_id.value).leader === member_id.value) {
            ElMessage.error("添加失败：要添加的员工是所属部门的主管！")
            ifValid = false
        }
        if (member_position.value === '2') {
            if (props.department_map.get(top_department_id.value).leader !== "") {
                ElMessage.error("添加失败：所属部门已经有主管！")
                ifValid = false
            }
        }

        if (ifValid) {
            emit('addPosition', member_name.value, member_id.value, member_position.value, top_department_name.value, top_department_id.value)
            ElMessage.success("添加成功！")
        }
    }

    top_department_id.value = ""
    top_department_name.value = ""
    member_name.value = ""
    member_id.value = ""
    member_position.value = '0'
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