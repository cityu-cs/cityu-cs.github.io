<script setup>
// https://stackoverflow.com/questions/10014271/generate-random-color-distinguishable-to-humans
function selectColor(number) {
    const hue = number * 137.508; // use golden angle approximation
    return `hsl(${hue},50%,75%)`;
}

const day2number = {
    Monday: 1,
    Tuesday: 2,
    Wednesday: 3,
    Thursday: 4,
    Friday: 5,
    Saturday: 6,
    Sunday: 7,
    Unknown: 0
};

const number2char = {
    1: 'M',
    2: 'T',
    3: 'W',
    4: 'R',
    5: 'F',
    6: 'S',
    7: 'U',
    0: 'X'
};

import { ref, onMounted, watch } from 'vue';
import courseDetail from '@/assets/newCourseDetail.json';

const rawDetail = ref([]);
const timetableList = ref([]);

function updateTimetableList(courseCode) {
    rawDetail.value = courseDetail[courseCode].sections;
    // 需要的数据：
    // rawDetail.sections.(any).activities
    // .day -> 0~7
    // .startTime -> 9:00~21:00
    // .endTime -> 9:50~23:50
    // .crn
    // .section

    for (const subsection of rawDetail.value) {
        for (const activity of subsection.activities) {
            console.log(activity);
        }
    }
}

function objectSpanMethod({ row, column, rowIndex, columnIndex }) {
    if (columnIndex === 0) {
        if (rowIndex % 2 === 0) {
            return {
                rowspan: 2,
                colspan: 1
            };
        } else {
            return {
                rowspan: 0,
                colspan: 0
            };
        }
    }
}

function tableCellStyle (row) {
    if (row.row[row.column.property].title !== undefined) {
        return 'background-color:rgb(24 144 255 / 80%);color: #fff; border-radius:10px'
    }
}

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