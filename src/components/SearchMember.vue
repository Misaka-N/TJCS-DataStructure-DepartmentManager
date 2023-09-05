<template>
    <div class="panel">
        <div class="title">查找人员</div>
        <div class="div-margin">
            <span class="sign-text">查找人员：</span>
            <el-select filterable remote v-model="member_name"
                       style="width: 200px;"
                       placeholder="请输入要查找的人员"
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
                   style="width: 150px;height: 30px;margin-top: 20px;margin-left: 65px" @click="searchDepartment"
        >查询人员
        </el-button>

    </div>
</template>

<script lang="ts" setup>
import {reactive, ref} from 'vue'
import {Search} from '@element-plus/icons-vue'
import {ElMessage} from "element-plus";

const props = defineProps(['member_map'])
const emit = defineEmits(['showResult'])

/* 定义内部用变量 */
interface ListItem {
    value: string
    label: string
}
const member_name = ref("")     // 要查找的部门名称
const loading = ref(false)          // 查找部门时，是否显示正在搜索
const options = ref<ListItem[]>([]) // 查找到的可选部门
const list = ref<ListItem[]>([])    // 搜索部门列表
let now_member = {
    member_name: '',        // 员工的姓名
    member_id: '',          // 员工的id
    member_position: []
}     // 要编辑的人员

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
}

const searchDepartment=()=>{
    if(member_name.value === '')
    {
        ElMessage.error('请输入人员姓名！')
        return
    }
    let result_member = props.member_map.get(now_member.member_id)
    emit('showResult', false, result_member)
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