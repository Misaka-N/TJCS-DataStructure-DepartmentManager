<template>
    <a class="back-button" href="index.html">
        <ArrowLeft style="width: 30px; height: 30px; margin-left: 15px;margin-top: 15px"/>
    </a>
    <section>
        <div class="code-table">
            <ul class="operation-box">
                <li v-if="operation===0" class="chosen" @click="changeOperation(0)">添加</li>
                <li v-else @click="changeOperation(0)">添加</li>
                <li v-if="operation===1" class="chosen" @click="changeOperation(1)">删除</li>
                <li v-else @click="changeOperation(1)">删除</li>
                <li v-if="operation===2" class="chosen" @click="changeOperation(2)">修改</li>
                <li v-else @click="changeOperation(2)">修改</li>
                <li v-if="operation===3" class="chosen" @click="changeOperation(3)">查找</li>
                <li v-else @click="changeOperation(3)">查找</li>
            </ul>
            <section class="operation-panel">
                <AddDepartment v-if="operation===0"
                               :name="props.name"
                               :id="origin_department_id"
                               :department_map="department_map"
                               @addDepartment="addDepartment"></AddDepartment>
                <DeleteDepartment v-else-if="operation===1"
                                  :department_map="department_map"
                                  :ori_department="origin_department_id"
                                  :member_map="member_map"
                                  @deleteDepartment="deleteDepartment"></DeleteDepartment>
                <EditDepartment v-else-if="operation===2"
                                :department_map="department_map"
                                :ori_department="origin_department_id"></EditDepartment>
                <SearchDepartment v-else-if="operation===3"
                                  :department_map="department_map"
                                  @showResult="showResult"></SearchDepartment>

                <AddMember v-if="operation===0"
                           :name="props.name"
                           :id="origin_department_id"
                           :member_map="member_map"
                           :department_map="department_map"
                           @addMember="addMember"></AddMember>
                <DeleteMember v-else-if="operation===1"
                              :department_map="department_map"
                              :member_map="member_map"
                              @deleteMember="deleteMember"></DeleteMember>
                <EditMember v-else-if="operation===2"
                            :member_map="member_map"></EditMember>
                <SearchMember v-else-if="operation===3"
                              :member_map="member_map"
                              @showResult="showResult"></SearchMember>


                <AddPosition v-if="operation===0"
                             :name="props.name"
                             :id="origin_department_id"
                             :member_map="member_map"
                             :department_map="department_map"
                             @addPosition="addPosition"></AddPosition>
                <DeletePosition v-else-if="operation===1"
                                :department_map="department_map"
                                :member_map="member_map"
                                @deletePosition="deletePosition"></DeletePosition>
            </section>
        </div>
        <div class="tree-table">
            <el-empty v-if="ifEmpty" description="暂无数据" :image-size="300" style="height: 750px"/>
            <Relationship v-else :head="props.origin_department_id" :department_map="department_map"
                          :member_map="member_map"></Relationship>
            <div v-if="ifShowResult" :class="ifDepartment ? 'result-panel-enhanced':'result-panel'">
                <el-icon class="panel-button" @click="closeResult">
                    <Close/>
                </el-icon>
                <div class="panel-sign-text">查询结果</div>
                <section v-if="ifDepartment" class="panel-section">
                    <div class="panel-sub-section">
                        <div class="panel-text"><span
                            class="panel-bold">部门名称：</span>{{ department_result.department_name }}
                        </div>
                        <div class="panel-text"><span
                            class="panel-bold">部门编号：</span>{{ department_result.department_id }}
                        </div>
                        <div class="panel-text"><span class="panel-bold">所属部门：</span>{{
                                department_result.top_department !== '' ? department_map.get(department_result.top_department).department_name : '暂无'
                            }}
                        </div>
                    </div>
                    <div class="panel-sub-section">
                        <div class="panel-text"><span
                            class="panel-bold">下属部门：</span>{{
                                department_result.sub_department.length === 0 ? '暂无' : department_map.get(department_result.sub_department[0]).department_name + '等' + department_result.sub_department.length + '个'
                            }}
                        </div>
                        <div class="panel-text"><span
                            class="panel-bold">主管：</span>{{
                                department_result.leader === '' ? '暂无' : member_map.get(department_result.leader).member_name
                            }}
                        </div>
                        <div class="panel-text"><span
                            class="panel-bold">副主管：</span>{{
                                department_result.vice_leader.length === 0 ? '暂无' : member_map.get(department_result.vice_leader[0]).member_name + '等' + department_result.vice_leader.length + '个'
                            }}
                        </div>
                    </div>
                    <el-table :data="table_department" style="width: 960px" stripe
                              max-height="250">
                        <el-table-column fixed prop="name" label="下属部门" width="480"/>
                        <el-table-column prop="id" label="部门编号" width="480"/>
                    </el-table>
                    <el-table :data="table_position" style="width: 960px;margin-top: 20px" stripe
                              max-height="250">
                        <el-table-column fixed prop="name" label="人员姓名" width="320"/>
                        <el-table-column prop="id" label="人员编号" width="320"/>
                        <el-table-column prop="position" label="人员职位" width="320"/>
                    </el-table>
                </section>
                <section v-else class="panel-section">
                    <div class="panel-sub-section">
                        <div class="panel-text"><span
                            class="panel-bold">人员名称：</span>{{ member_result.member_name }}
                        </div>
                        <div class="panel-text"><span
                            class="panel-bold">人员编号：</span>{{ member_result.member_id }}
                        </div>
                        <div class="panel-text"><span class="panel-bold">所属部门：</span>{{
                                department_map.get(member_result.member_position[0].department_id).department_name + '等' + member_result.member_position.length + '个'
                            }}
                        </div>
                    </div>
                    <el-table :data="table_member" style="width: 960px" stripe max-height="250">
                        <el-table-column fixed prop="name" label="所在部门" width="320"/>
                        <el-table-column prop="id" label="部门编号" width="320"/>
                        <el-table-column prop="position" label="人员职位" width="320"/>
                    </el-table>
                </section>
            </div>

        </div>
    </section>
</template>

<script setup>
/* 引入必要的组件和文件 */
import {onMounted, reactive, ref} from 'vue'
import {ArrowLeft, Close} from '@element-plus/icons-vue'
import AddDepartment from "@/components/AddDepartment.vue";
import Relationship from "@/components/Relationship.vue";
import DeleteDepartment from "@/components/DeleteDepartment.vue";
import EditDepartment from "@/components/EditDepartment.vue"
import {ElMessage} from "element-plus";
import SearchDepartment from "@/components/SearchDepartment.vue";
import AddMember from "@/components/AddMember.vue";
import AddPosition from "@/components/AddPosition.vue";
import DeleteMember from "@/components/DeleteMember.vue";
import DeletePosition from "@/components/DeletePosition.vue";
import EditMember from "@/components/EditMember.vue";
import SearchMember from "@/components/SearchMember.vue";


const props = defineProps(['name', 'origin_department_id'])

/* 定义内部用的变量 */
const ifEmpty = ref(true)
const ifShowResult = ref(false)     // 是否展示查询结果
const ifDepartment = ref(false)     // 展示结果是否为部门数据
let department_result = {
    department_id: "",      // 机构id
    top_department: "",      // 所属机构
    sub_department: [],      // 下属机构
    department_name: "",     // 机构名称
    leader: "",             // 主管id（1名）
    vice_leader: [],        // 副主管id（多名）
    members: []              // 办事人员
}
let member_result = {
    member_name: '',        // 员工的姓名
    member_id: '',          // 员工的id
    member_position: []
}
let table_member = []
let table_department = []
let table_position = []
const operation = ref(0)             // 当前对数据进行得是何种操作
let department_map = reactive(new Map([[props.origin_department_id, {// 存储组织机构
    department_id: props.origin_department_id,      // 机构id
    top_department: "",      // 所属机构
    sub_department: [],      // 下属机构
    department_name: props.name,     // 机构名称
    leader: "",             // 主管id（1名）
    vice_leader: [],        // 副主管id（多名）
    members: []              // 办事人员
}
]]))        // 建立id和机构的映射


let member_map = reactive(new Map)  // 建立员工id和员工的映射
/*  员工对象的定义如下：
    member_name: '',        // 员工的姓名
    member_id: '',          // 员工的id
    member_position: [],    // 员工的各种职位，列表中的每个元素都包含一个对象{department_id, position}
* */


const changeOperation = (index) => {
    operation.value = index
}

const addDepartment = (department_name, department_id, top_department_name, top_department_id) => {
    let top_department = department_map.get(top_department_id)
    top_department.sub_department.push(department_id)
    department_map.set(department_id, {
        department_id: department_id,
        top_department: top_department_id,
        sub_department: [],
        department_name: department_name,
        leader: "",
        vice_leader: [],
        members: []
    })
    ifEmpty.value = false
}

const deleteDepartment = (new_department_map, new_member_map) => {
    department_map = new_department_map
    member_map = new_member_map
    if (department_map.size < 2)
        ifEmpty.value = true
    console.log(member_map)
}


const showResult = (ifDepartmentFlag, result) => {
    ifShowResult.value = false
    ifDepartment.value = ifDepartmentFlag
    if (ifDepartment.value) {
        department_result = result
        for (let i = 0; i < department_result.sub_department.length; i++) {
            let sub_department = department_map.get(department_result.sub_department[i])
            table_department.push({
                name: sub_department.department_name,
                id: sub_department.department_id
            })
        }
        if (department_result.leader !== '') {
            let now_member = member_map.get(department_result.leader)
            table_position.push({
                name: now_member.member_name,
                id: now_member.member_id,
                position: '主管'
            })
        }
        if (department_result.vice_leader.length !== 0) {
            for (let i = 0; i < department_result.vice_leader.length; i++) {
                let now_member = member_map.get(department_result.vice_leader[i])
                table_position.push({
                    name: now_member.member_name,
                    id: now_member.member_id,
                    position: '副主管'
                })
            }
        }
        if (department_result.members.length !== 0) {
            for (let i = 0; i < department_result.members.length; i++) {
                let now_member = member_map.get(department_result.members[i])
                table_position.push({
                    name: now_member.member_name,
                    id: now_member.member_id,
                    position: '办事人员'
                })
            }
        }
    } else {
        member_result = result
        for (let i = 0; i < member_result.member_position.length; i++) {
            let now_department = department_map.get(member_result.member_position[i].department_id)
            table_member.push({
                name: now_department.department_name,
                id: now_department.department_id,
                position: member_result.member_position[i].position === '0' ? '办事人员' :
                    member_result.member_position[i].position === '1' ? '副主管' : '主管'
            })
        }
    }
    ifShowResult.value = true
}

const closeResult = () => {
    ifShowResult.value = false
    ifDepartment.value = false
    department_result = {
        department_id: "",      // 机构id
        top_department: "",      // 所属机构
        sub_department: [],      // 下属机构
        department_name: "",     // 机构名称
        leader: "",             // 主管id（1名）
        vice_leader: [],        // 副主管id（多名）
        members: []              // 办事人员
    }
    member_result = {
        member_name: '',        // 员工的姓名
        member_id: '',          // 员工的id
        member_position: []
    }
    table_member = []
    table_department = []
    table_position = []
}

const addMember = (member_name, member_id, member_position, top_department_name, top_department_id) => {
    let top_department = department_map.get(top_department_id)
    if (member_position === '0')
        top_department.members.push(member_id)
    else if (member_position === '1')
        top_department.vice_leader.push(member_id)
    else
        top_department.leader = member_id

    member_map.set(member_id, {
        member_name: member_name,        // 员工的姓名
        member_id: member_id,          // 员工的id
        member_position: [{
            department_id: top_department_id,
            position: member_position
        }]
    })
    ifEmpty.value = false
}

const addPosition = (member_name, member_id, member_position, top_department_name, top_department_id) => {
    let top_department = department_map.get(top_department_id)
    if (member_position === '0')
        top_department.members.push(member_id)
    else if (member_position === '1')
        top_department.vice_leader.push(member_id)
    else
        top_department.leader = member_id

    member_map.get(member_id).member_position.push({
        department_id: top_department_id,
        position: member_position
    })
    ifEmpty.value = false
}

const deleteMember = (new_department_map, new_member_map) => {
    department_map = new_department_map
    member_map = new_member_map
    if (department_map.size < 2 && member_map.size === 0)
        ifEmpty.value = true
}

const deletePosition = (new_department_map, new_member_map) => {
    department_map = new_department_map
    member_map = new_member_map
    if (department_map.size < 2 && member_map.size === 0)
        ifEmpty.value = true
}
</script>

<style scoped>
@keyframes fadeIn {
    0% {
        opacity: 0;
    }
    100% {
        opacity: 1;
    }
}

section {
    display: flex;
    justify-content: center;
    width: 90%;
    height: 80%;
    position: absolute;
    top: 13%;
    left: 5%;
    animation: fadeIn 1s forwards;
    animation-fill-mode: both;
    animation-duration: 1s;
}

.back-button {
    width: 60px;
    height: 60px;
    border-radius: 1000px;
    background: rgba(255, 255, 255, 0.6);
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
    margin-top: 5%;
    position: absolute;
    bottom: 1.5%;
    right: 1.5%;
    animation: fadeIn 1s forwards;
    animation-fill-mode: both;
    animation-duration: 1s;
    transition: box-shadow 0.5s ease;
}

.back-button:hover {
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.4);
}

.code-table {
    width: 320px;
    height: 750px;
    border-radius: 10px;
    background: rgba(255, 255, 255, 1);
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}

.operation-box {
    width: 280px;
    height: 40px;
    display: flex;
    justify-content: space-between;
    margin-left: 20px;
    margin-right: 20px;
    font-family: Msyh;
    letter-spacing: 2px;
}

.operation-box li {
    padding-top: 15px;
    display: inline-block;
    width: 40px;
    border-bottom: 2px solid #B2D7D2;
    text-align: center;
    color: #888888;
    font-size: 14px;
    transition: color 0.5s ease;
    cursor: default;
}

.operation-box li:hover {
    color: #B2D7D2;
}

.operation-box .chosen {
    color: #B2D7D2;
}

.tree-table {
    width: 1120px;
    height: 750px;
    border-radius: 10px;
    background: rgba(255, 255, 255, 0.6);
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
    margin-left: 30px;
    position: relative;
    z-index: -1;
}

.operation-panel {
    margin-left: -25px;
    margin-top: -40px;
    width: 280px;
    height: 680px;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
//border: solid 1px black;
}

.result-panel {
    width: 1100px;
    height: 300px;
    position: absolute;
    background-color: rgba(255, 255, 255, 0.8);
    border-radius: 10px;
    top: 10px;
    left: 10px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}

.result-panel-enhanced {
    width: 1100px;
    height: 550px;
    position: absolute;
    background-color: rgba(255, 255, 255, 0.8);
    border-radius: 10px;
    top: 10px;
    left: 10px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);
}


.panel-button {
    position: absolute;
    top: 10px;
    right: 10px;
}

.panel-button:hover {
    cursor: pointer;
}

.panel-section {
    display: flex;
    width: 1100px;
    flex-direction: column;
}

.panel-sign-text {
    font-family: Msyh;
    text-align: center;
    width: 1100px;
    font-size: 20px;
    letter-spacing: 2px;
    color: #555555;
    margin-top: 10px;
    margin-bottom: 10px;
}

.panel-bold {
    font-size: 18px;
    font-weight: bold;
}

.panel-sub-section {
    display: flex;
}

.panel-sub-section:first-child {
    margin-top: 20px;
}

.panel-text {
    padding-top: 10px;
    width: 340px;
    height: 50px;
    margin-top: 20px;
    font-size: 18px;
}


</style>