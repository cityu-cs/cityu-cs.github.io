<script setup>
import { ref, computed } from 'vue';
import courses from '@/assets/newCourseList.json';
import courseDetail from '@/assets/newCourseDetail.json';

const DEBUG = true;

/* PART 1: 课程列表 */

const fullList = ref(courses.course);

const subjectList = ref(courses.subject.map(subject => {
    return {
        value: subject,
        label: subject
    };
}));

const levelList = ref([
    {value: 'A', label: 'Associate Degree'},
    {value: 'B', label: 'Bachelor\'s Degree'},
    {value: 'P', label: 'Postgraduate Degree'},
    {value: 'D', label: 'Professional Doctorate'},
    {value: 'R', label: 'Research Degree'}
]);

const char2Day = {
    'M': 'Mon',
    'T': 'Tue',
    'W': 'Wed',
    'R': 'Thu',
    'F': 'Fri',
    'S': 'Sat',
    'U': 'Sun',
    'X': 'TBA'
}

const queryText = ref('');
const querySubject = ref('');
const queryLevel = ref('');
const resultList = ref([]);
const currentList = ref([]);

function search() {
    const query = queryText.value.toLowerCase();
    const subject = querySubject.value;
    const level = queryLevel.value;

    if (!query && !subject && !level) {
        resultList.value = fullList.value;
        total.value = resultList.value.length;
        updateCurrentList();
        return;
    }

    resultList.value = fullList.value.filter(course => {
        if (query && !course.courseCode.toLowerCase().includes(query) && !course.courseTitle.toLowerCase().includes(query)) {
            return false;
        }
        if (subject && course.subject !== subject) {
            return false;
        }
        if (level && !course.courseLevel.includes(level)) {
            return false;
        }
        return true;
    });

    total.value = resultList.value.length;
    updateCurrentList();
}

const pagesize = ref(10);
const currentPage = ref(1);
const total = ref(0);
function handleSizeChange(val) {
    pagesize.value = val;
    currentPage.value = 1;
    updateCurrentList();
}
function handleCurPageChange(val) {
    currentPage.value = val;
    updateCurrentList();
}

function updateCurrentList() {
    const start = (currentPage.value - 1) * pagesize.value;
    const end = start + pagesize.value;
    currentList.value = resultList.value.slice(start, end);
}

/* PART 2: 课程详情 */

const tableItems = ref({});
const tabs = ref([]);
const restrictions = ref({});
const activeTab = ref('');
const sectionStatus = ref({});
const updateKey = ref(0);

function getDetail(courseCode) {
    // category 是一个 Object[]，所以用 map 取出第一个键值
    tabs.value = courseDetail[courseCode].category.map(obj => Object.keys(obj)[0]);
    activeTab.value = tabs.value[0];
    for (let tab of tabs.value) {
        tableItems.value[tab] = courseDetail[courseCode].activities.filter(activity => {
            return activity.category === tab && activity.day !== 'X';
        });
        restrictions.value[tab] = courseDetail[courseCode].category[tab];
        if (restrictions.value[tab] !== undefined) {
            restrictions.value[tab] = restrictions.value[tab][0];
        }
    }
    for (let tab of tabs.value) {
        for (let activity of tableItems.value[tab]) {
            activity.dayTime = char2Day[activity.day] + ' ' + activity.startTime + '-' + activity.endTime;
            activity.buildingRoom = activity.bldg + ' ' + activity.room;
            activity.webEnabled = activity.web ? 'Y' : 'N';
            activity.startHour = parseInt(activity.startTime.split(':')[0]);
            activity.endHour = parseInt(activity.endTime.split(':')[0]);
            activity.courseCode = courseCode;
        }
    }
    getAllStatus();
}

function handleTabClick(tab, event) {
    activeTab.value = tab.props.name;
    getAllStatus();
}

function handleCatalogClick(courseCode) {
    window.open(`http://www.cityu.edu.hk/catalogue/ug/current/course/${courseCode}.htm`, '_blank');
}

function getAllStatus() {
    if (activeTab.value === '') {
        return;
    }
    for (let act of tableItems.value[activeTab.value]) {
        for (let added of timetableList.value) {
            if (added.crn === act.crn) {
                sectionStatus.value[act.crn] = 'Added';
                break;
            }
            if (added.courseCode === act.courseCode && added.category !== act.category) {
                sectionStatus.value[act.crn] = 'Incompatible';
                break;
            }
            if (added.day === act.day && added.endHour >= act.startHour && added.startHour <= act.endHour) {
                sectionStatus.value[act.crn] = 'Clash';
                break;
            }
        }
        if (sectionStatus.value[act.crn] === undefined) {
            sectionStatus.value[act.crn] = 'OK';
        }
    }
    updateKey.value += 1;
}

/* PART 3: 课程表 */

const hours = [9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22];
const dayKeys = ['M', 'T', 'W', 'R', 'F', 'S', 'U'];
const timetableList = ref([]);

function saveLocalStorage() {
    localStorage.setItem('timetableList', JSON.stringify(timetableList.value));
}

function loadLocalStorage() {
    const data = localStorage.getItem('timetableList');
    if (data) {
        timetableList.value = JSON.parse(data);
    }
}

function resetTimetable() {
    if (!confirm('Reset Timetable?')) {
        return;
    }
    localStorage.removeItem('timetableList');
    // 强制刷新页面
    window.location.reload();
}

function addToTimetable(row) {
    timetableList.value.push(row);
    getAllStatus();
    saveLocalStorage();
}

function mergeTimetable({row, column, rowIndex, columnIndex}) {
    if (columnIndex === 0) {
        return {rowspan: 1, colspan: 1};
    }
    for (let act of timetableList.value) {
        /* FIXME: 合并单元格存在 bug。比如，新建周三 9-11 点的课程时，周四 10-10 点的课程会因为周三 10-10 点这一单元格被占用
                  而向右移动到周五 10-10 点。
         */
        if (act.day === dayKeys[columnIndex - 1] && act.startHour === row) {
            return {rowspan: act.endHour - act.startHour + 1, colspan: 1};
        }
    }
    return {rowspan: 1, colspan: 1};
}

// https://stackoverflow.com/questions/10014271/generate-random-color-distinguishable-to-humans
function selectColor(number) {
    let hue = number * 137.508; // use golden angle approximation
    return `hsl(${hue},50%,60%)`;
}

function tableCellStyle ({row, column, rowIndex, columnIndex}) {
    if (columnIndex === 0) {
        return {'padding': '0', 'text-align': 'center'};
    }
    for (let act of timetableList.value) {
        if (act.day === dayKeys[columnIndex - 1] && act.startHour === row) {
            return {'background-color': selectColor(parseInt(act.courseCode.slice(2, 5))),
                'color': '#fff', 'border-radius': '5px', 'padding': '0'};
        } // 不同 courseCode 对应不同颜色
    }
    return {'padding': '0'};
}

function deleteFromTimetable(row, column, cell, event) {
    if (column.minWidth === 4) {
        for (let i = 0; i < timetableList.value.length; i++) {
            if (timetableList.value[i].day === column.rawColumnKey && timetableList.value[i].startHour === row) {
                if (!confirm(`Delete ${timetableList.value[i].courseCode} - ${timetableList.value[i].section}?`)) {
                    return;
                }
                timetableList.value.splice(i, 1);
                saveLocalStorage();
                break;
            }
        }
    }
    getAllStatus();
}

function deleteFromDetail(row) {
    if (!confirm(`Delete ${row.courseCode} - ${row.section}?`)) {
        return;
    }
    for (let i = 0; i < timetableList.value.length; i++) {
        if (timetableList.value[i].crn === row.crn) {
            timetableList.value.splice(i, 1);
            saveLocalStorage();
            break;
        }
    }
    getAllStatus();
}

/* PART 4: 初始化 */
search();
loadLocalStorage();

</script>

<template>

    <!-- 课程列表 -->

    <div class="search-container">
        <!-- 当用户在文本框中输入时，按下回车键时触发搜索 -->
        <el-input class="search-input" placeholder="Course Code or Title" v-model="queryText" clearable
            @keyup.enter.native="search"></el-input>
        <el-select class="search-select" v-model="querySubject" placeholder="Subject" clearable>
            <el-option v-for="item in subjectList" :key="item.value" :label="item.label" :value="item.value"></el-option>
        </el-select>
        <el-select class="search-select" v-model="queryLevel" placeholder="Level" clearable>
            <el-option v-for="item in levelList" :key="item.value" :label="item.label" :value="item.value"></el-option>
        </el-select>
        <el-button class="search-button" type="primary" @click="search">Search</el-button>
    </div>

    <div class="table-container">
        <el-table
                :data="currentList"
                border
                stripe
                :header-cell-style="{background:'#d9e5fd', color:'black', fontWeight: 1000}"
                :row-style="{height: '0'}"
                :cell-style="{padding: '0'}"
        >
            <el-table-column class="table-medium" prop="courseCode" label="Course Code" min-width="2"></el-table-column>
            <el-table-column class="table-long" prop="courseTitle" label="Course Title" min-width="4"></el-table-column>
            <el-table-column class="table-short" prop="creditUnits" label="Credit Units" min-width="1"></el-table-column>
            <el-table-column class="table-short" prop="webEnabled" label="Web Enabled" min-width="1"></el-table-column>
            <el-table-column class="table-medium" prop="action" label="Action" min-width="2">
                <template #default="{row}">
                    <el-button type="text" @click="handleCatalogClick(row.courseCode)">Catalog</el-button>
                    <el-button type="text" @click="getDetail(row.courseCode)">Sections</el-button>
                </template>
            </el-table-column>
        </el-table>
        <div class="pagination-container">
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurPageChange"
                :current-page="currentPage"
                :page-sizes="[10, 20, 50, 100]"
                :page-size="pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>
        </div>
    </div>

    <!-- 课程详情 -->

    <div>
        <div class="tab-container">
            <div class="flex-container">
                <div class="flex-item-left">
                <el-tabs value="activeTab" @tab-click="handleTabClick" :type="'card'">
                    <el-tab-pane v-for="item in tabs" :key="item" :label="item" :name="item"></el-tab-pane>
                </el-tabs>
                </div>
                <div class="flex-item-right">
                    <el-button type="danger" @click="resetTimetable">Reset Timetable</el-button>
                </div>
            </div>
            <p v-if="restrictions[activeTab] !== undefined">Restriction:</p>
            <ul>
                <li v-for="item in restrictions[activeTab]" :key="item">{{item}}</li>
            </ul>
            <p>Instruction:</p>
            <ul>
                <li>Use the tab above to navigate between categories. You can only choose sections from the same category.</li>
                <li>Click on a section in the section table to add or remove it from the timetable.</li>
                <li>Click on a section in the timetable to remove it.</li>
            </ul>
            <div class="table-container">
                <el-table
                        :data="tableItems[activeTab]"
                        border
                        stripe
                        :header-cell-style="{background:'#d9e5fd', color:'black', fontWeight: 1000}"
                        :row-style="{height: '0'}"
                        :cell-style="{padding: '0'}"
                        :key="updateKey"
                >
                    <el-table-column class="table-short" prop="crn" label="CRN" min-width="1"></el-table-column>
                    <el-table-column class="table-short" prop="section" label="Section" min-width="1"></el-table-column>
                    <el-table-column class="table-medium" prop="dayTime" label="Time" min-width="2"></el-table-column>
                    <el-table-column class="table-medium" prop="buildingRoom" label="Venue" min-width="2"></el-table-column>
                    <el-table-column class="table-short" prop="webEnabled" label="Web" min-width="1"></el-table-column>
                    <el-table-column clash="table-short" prop="status" label="Status" min-width="1">
                        <template #default="{row}">
                            <p>{{sectionStatus[row.crn]}}</p>
                        </template>
                    </el-table-column>
                    <el-table-column class="table-medium" prop="add" label="Action" min-width="2">
                        <template #default="{row}">
                            <el-button type="text" @click="addToTimetable(row)" v-if="sectionStatus[row.crn] === 'OK'">Add</el-button>
                            <el-button type="text" v-else-if="sectionStatus[row.crn] === 'Added'" @click="deleteFromDetail(row)">Remove</el-button>
                            <el-button type="text" disabled v-else>{{sectionStatus[row.crn]}}</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </div>
        </div>
    </div>

    <!-- 课程表 -->

    <div class="timetable-container">
        <el-table
                :data="hours"
                border
                :header-cell-style="{background:'#d9e5fd', color:'black', fontWeight: 1000, textAlign: 'center'}"
                :span-method="mergeTimetable"
                :row-style="{height: '70px'}"
                :cell-style="tableCellStyle"
                @cell-click="deleteFromTimetable"
        >
            <el-table-column label="Time" min-width="2">
                <template #default="{row}">
                    <p>{{row}}:00</p>
                </template>
            </el-table-column>
            <el-table-column v-for="day in dayKeys" :key="day" :label="char2Day[day]" min-width="4">
                <template #default="{row}">
                    <div v-for="act in timetableList" :key="act.crn" v-show="act.day === day && act.startHour === row" class="timetable-data">
                        <p class="timetable-data-title">{{act.courseCode}} - {{act.section}}</p>
                        <p class="timetable-data-info">{{act.dayTime}}</p>
                        <p class="timetable-data-info">{{act.buildingRoom}}</p>
                    </div>
                </template>
            </el-table-column>
        </el-table>
    </div>

</template>

<style scoped>
.search-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    margin-bottom: 10px;
}

.search-input {
    width: 40%;
}

.search-select {
    width: 20%;
}

.search-button {
    width: 20%;
}

.table-container {
    width: 100%;
}

.table-medium {
    min-width: 20%;
}

.table-long {
    min-width: 40%;
}

.table-short {
    min-width: 10%;
}

.pagination-container {
    display: flex;
    justify-content: flex-end;
    margin-top: 10px;
}

.tab-container {
    margin-top: 20px;
}

.table-container {
    margin-top: 10px;
}

.timetable-container {
    width: 100%;
    margin-top: 20px;
}

.timetable-data-title {
    font-weight: bold;
}

.timetable-data-title, .timetable-data-info {
    text-align: center;
}

.flex-container {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-top: 10px;
}

.flex-item-right {
    right: 0;
}
</style>