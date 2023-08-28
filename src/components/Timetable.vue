<script setup>
// https://stackoverflow.com/questions/10014271/generate-random-color-distinguishable-to-humans
function selectColor(number) {
    const hue = number * 137.508; // use golden angle approximation
    return `hsl(${hue},50%,75%)`;
}

import { ref, onMounted, watch } from 'vue';
import courseDetail from '@/assets/newCourseDetail.json';

const rawDetail = ref([]);
const timetableList = ref([]);
const hours = [9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23];

function updateTimetableList(courseCode) {
    rawDetail.value = courseDetail[courseCode];
    for (const activity of rawDetail.value.activities) {
        console.log(activity);
    }
}


function objectSpanMethod({ row, column, rowIndex, columnIndex }) { // 多行合并
    switch (columnIndex) {
        case 1:
            if (row.M.courseCode !== undefined) {
                return {rowspan: row.M.end - row.M.start + 1, colspan: 1};
            }
            break;
        case 2:
            if (row.T.courseCode !== undefined) {
                return {rowspan: row.T.end - row.T.start + 1, colspan: 1};
            }
            break;
        case 3:
            if (row.W.courseCode !== undefined) {
                return {rowspan: row.W.end - row.W.start + 1, colspan: 1};
            }
            break;
        case 4:
            if (row.R.courseCode !== undefined) {
                return {rowspan: row.R.end - row.R.start + 1, colspan: 1};
            }
            break;
        case 5:
            if (row.F.courseCode !== undefined) {
                return {rowspan: row.F.end - row.F.start + 1, colspan: 1};
            }
            break;
        case 6:
            if (row.S.courseCode !== undefined) {
                return {rowspan: row.S.end - row.S.start + 1, colspan: 1};
            }
            break;
        case 7:
            if (row.U.courseCode !== undefined) {
                return {rowspan: row.U.end - row.U.start + 1, colspan: 1};
            }
            break;
    }
    return {rowspan: 1, colspan: 1};
}

function tableCellStyle (row) {
    if (row.row[row.column.property].title !== undefined) {
        return 'background-color:rgb(24 144 255 / 80%);color: #fff; border-radius:10px'
    }
}

// test
updateTimetableList("AC3202");
</script>

<template>
    <!-- 以表格形式呈现课程列表 -->
    <!-- Monday ~ Sunday, 09:00 - 23:00 -->
    <!-- 注意，时间点应该在行与行的交界处，而不是行的中间 -->
    <!-- 如果暂时实现不了，可以显示 09 代替 09:00-09:50，小时数显示在正中间 -->
    <div class="timetable-container">
        <el-table :data="timetableList"
                  :span-method="objectSpanMethod"
                  border
                  :header-cell-style="{background:'#d9e5fd', color:'black', fontWeight: 1000}"
                  :cell-style="tableCellStyle"
                    style="width: 100%">
            <el-table-column prop="hour" label="Time" min-width="2"></el-table-column>
            <el-table-column prop="M" label="Monday" min-width="4">
                <template slot-scope="scope">
                    <h4>{{scope.row.M.courseCode}}</h4>
                    <div v-html="scope.row.M.info"></div>
                </template>
            </el-table-column>
            <el-table-column prop="T" label="Tuesday" min-width="4">
                <template slot-scope="scope">
                    <h4>{{scope.row.T.courseCode}}</h4>
                    <div v-html="scope.row.T.info"></div>
                </template>
            </el-table-column>
            <el-table-column prop="W" label="Wednesday" min-width="4">
                <template slot-scope="scope">
                    <h4>{{scope.row.W.courseCode}}</h4>
                    <div v-html="scope.row.W.info"></div>
                </template>
            </el-table-column>
            <el-table-column prop="R" label="Thursday" min-width="4">
                <template slot-scope="scope">
                    <h4>{{scope.row.R.courseCode}}</h4>
                    <div v-html="scope.row.R.info"></div>
                </template>
            </el-table-column>
            <el-table-column prop="F" label="Friday" min-width="4">
                <template slot-scope="scope">
                    <h4>{{scope.row.F.courseCode}}</h4>
                    <div v-html="scope.row.F.info"></div>
                </template>
            </el-table-column>
            <el-table-column prop="S" label="Saturday" min-width="4">
                <template slot-scope="scope">
                    <h4>{{scope.row.S.courseCode}}</h4>
                    <div v-html="scope.row.S.info"></div>
                </template>
            </el-table-column>
            <el-table-column prop="U" label="Sunday" min-width="4">
                <template slot-scope="scope">
                    <h4>{{scope.row.U.courseCode}}</h4>
                    <div v-html="scope.row.U.info"></div>
                </template>
            </el-table-column>
        </el-table>
    </div>
</template>

<style scoped>
.timetable-container {
    width: 100%;
    margin-top: 20px;
}
</style>