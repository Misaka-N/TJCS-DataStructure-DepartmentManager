<template>
    <div class="panel">
        <div class="title">修改人员</div>
        <div class="div-margin">
            <span class="sign-text">修改人员：</span>
            <el-select filterable remote v-model="member_name"
                       style="width: 200px;"
                       placeholder="请输入要修改的人员"
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
            <span class="sign-text">人员姓名：</span>
            <el-input
                v-model="new_member_name"
                :disabled="ifDisable"
                style="width: 200px;"></el-input>
        </div>
        <el-button :icon="Edit" round plain color="#76B6EA" size="large"
                   style="width: 150px;height: 30px;margin-top: 20px;margin-left: 65px" @click="editMember"
        >修改人员
        </el-button>

    </div>
</template>

<script lang="ts" setup>
import {reactive, ref} from 'vue'
import {Edit} from '@element-plus/icons-vue'
import {ElMessage} from "element-plus";

const props = defineProps(['member_map'])

/* 定义内部用变量 */
interface ListItem {
    value: string
    label: string
}

const ifDisable = ref(true)
const member_name = ref("")     // 要修改的部门名称
const new_member_name = ref("") // 修改的部门名称
let now_member = {
    member_name: '',        // 员工的姓名
    member_id: '',          // 员工的id
    member_position: []
}     // 要编辑的人员
const loading = ref(false)          // 查找部门时，是否显示正在搜索
const options = ref<ListItem[]>([]) // 查找到的可选部门
const list = ref<ListItem[]>([])    // 搜索部门列表


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
    let member_id = member_name.value.substring(i + 1, member_name.value.length - 1)
    now_member = {...props.member_map.get(member_id)}
    /* 给各个变量赋值 */
    ifDisable.value = false
    new_member_name.value = now_member.member_name
}


const editMember = () => {
    now_member.member_name = new_member_name.value
    props.member_map.delete(now_member.member_id)
    props.member_map.set(now_member.member_id, now_member)
    ElMessage.success("修改成功！")

    /* 清空变量 */
    ifDisable.value = true
    member_name.value = ""
    new_member_name.value = ""
    now_member = {
        member_name: '',        // 员工的姓名
        member_id: '',          // 员工的id
        member_position: []
    }     // 要编辑的部门
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