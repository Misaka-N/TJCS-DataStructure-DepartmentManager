<template>
    <div class="graph-panel">
        <el-scrollbar height="750px">
            <svg id="graph-svg"></svg>
        </el-scrollbar>
        <div class="tree-results">
            若关系图太大可通过鼠标滚轮上下移动图片
        </div>
    </div>
</template>

<script setup>
import * as d3 from 'd3'
import {onMounted, watch} from "vue";

const props = defineProps(['head', 'department_map', 'member_map'])
let myData = {
    name: props.department_map.get(props.head).department_name,
    children: []
}

const generateData = (fatherDepartment, nowDepartment, nowData) => {
    // console.log(fatherDepartment)
    // console.log(nowDepartment)
    // console.log(nowData)
    if (nowDepartment.sub_department.length === 0 && nowDepartment.leader === "" && nowDepartment.vice_leader.length === 0 && nowDepartment.members.length === 0) {
        nowData.push({name: nowDepartment.department_name})
    } else {
        nowData.push({name: nowDepartment.department_name, children: []})
        if (nowDepartment.leader !== "")
            nowData[nowData.length - 1].children.push({
                name: props.member_map.get(nowDepartment.leader).member_name,
                size: 3
            })
        for (let i = 0; i < nowDepartment.vice_leader.length; i++) {
            nowData[nowData.length - 1].children.push({
                name: props.member_map.get(nowDepartment.vice_leader[i]).member_name,
                size: 2
            })
        }
        for (let i = 0; i < nowDepartment.members.length; i++) {
            nowData[nowData.length - 1].children.push({
                name: props.member_map.get(nowDepartment.members[i]).member_name,
                size: 1
            })
        }

        for (let i = 0; i < nowDepartment.sub_department.length; i++) {
            generateData(nowDepartment, props.department_map.get(nowDepartment.sub_department[i]), nowData[nowData.length - 1].children)
        }
    }
}

const generateGraph = () => {
    let headDepartment = props.department_map.get(props.head)
    myData = {
        name: headDepartment.department_name,
        children: []
    }
    d3.select('#graph-svg').selectAll('*').remove();

    // console.log(props.member_map)
    // console.log(props.department_map)


    if (headDepartment.leader !== "")
        myData.children.push({name: props.member_map.get(headDepartment.leader).member_name, size: 3})
    for (let i = 0; i < headDepartment.vice_leader.length; i++) {
        myData.children.push({name: props.member_map.get(headDepartment.vice_leader[i]).member_name, size: 2})
    }
    for (let i = 0; i < headDepartment.members.length; i++) {
        myData.children.push({name: props.member_map.get(headDepartment.members[i]).member_name, size: 1})
    }

    for (let i = 0; i < props.department_map.get(props.head).sub_department.length; i++) {
        generateData(props.department_map.get(props.head), props.department_map.get(props.department_map.get(props.head).sub_department[i]), myData.children)
    }


    // console.log(myData)

    let width = 1120;

    let root = d3.hierarchy(myData);
    let dx = 20;
    let dy = width / (root.height + 1);

    let tree = d3.tree().nodeSize([dx, dy]);

    root.sort((a, b) => {
        const aIsSpecial = 'size' in a.data;
        const bIsSpecial = 'size' in b.data;

        if (aIsSpecial && !bIsSpecial) return 1;
        if (!aIsSpecial && bIsSpecial) return -1;
        if (aIsSpecial && bIsSpecial) {
            if (a.data.size === b.data.size)
                return d3.ascending(a.data.name, b.data.name);
            else
                return a.data.size > b.data.size ? -1 : 1
        }

        return d3.ascending(a.data.name, b.data.name);
    });

    tree(root);

    let x0 = Infinity;
    let x1 = -x0;
    root.each(d => {
        if (d.x > x1) x1 = d.x;
        if (d.x < x0) x0 = d.x;
    });

    let height = x1 - x0 + dx * 2;

    let svg = d3.select("#graph-svg")
        .attr("width", width)
        .attr("height", Math.max(750, height))   // 这样在高度小于750时可以保证居中
        .attr("viewBox", [-dy / 3, x0 - dx, width, height])
        .attr("style", "max-width: 100%; height: auto; font: 10px sans-serif;");

    let link = svg.append("g")
        .attr("fill", "none")
        .attr("stroke-opacity", 0.4)
        .selectAll()
        .data(root.links())
        .join("path")
        .attr("stroke", d => {
            if (d.target.data.size) {
                if (d.target.data.size === 3)
                    return "#FF6F61";
                else if (d.target.data.size === 2)
                    return "#FFCF48"
                else
                    return "#76B6EA";
            } else {
                return "#555";
            }
        })
        .attr("d", d3.linkHorizontal()
            .x(d => d.y)
            .y(d => d.x))
        .attr("stroke-width", d => {
            if (d.target.data.size) {
                return 1.5;
            } else {
                return 2;
            }
        });

    let node = svg.append("g")
        .attr("stroke-linejoin", "round")
        .attr("stroke-width", 3)
        .selectAll()
        .data(root.descendants())
        .join("g")
        .attr("transform", d => `translate(${d.y},${d.x})`);


    node.append("circle")
        .attr("fill", d => {
            if (d.children) {
                return "#555";
            } else if (d.data.size) {
                if (d.data.size === 3)
                    return "#FF6F61";
                else if (d.data.size === 2)
                    return "#FFCF48"
                else
                    return "#76B6EA";
            } else {
                return "#999";
            }
        })
        .attr("r", 2.5);

    node.append("text")
        .attr("dy", "0.31em")
        .attr("x", d => {
            if (d.children) {
                return -6;
            } else {
                return 6;
            }
        })
        .attr("text-anchor", d => d.children ? "end" : "start")
        .text(d => d.data.name)
        .clone(true).lower()
        .attr("stroke", "white");
}

onMounted(() => {
    generateGraph()
})

watch(props.department_map, () => {
    generateGraph()
});

watch(props.member_map, () => {
    generateGraph()
});


</script>


<style scoped>
.graph-panel {
    width: 1120px;
    height: 750px;
    border-radius: 10px;
    z-index: 1;
}

#graph-svg {
    width: 1120px;
    z-index: 1;
}

.tree-results {
    color: #888888;
    font-size: 14px;
    font-family: STZhongsong;
    position: absolute;
    left: 10px;
    bottom: 10px;
}
</style>