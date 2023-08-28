<script setup>
import { ref } from 'vue';
import courses from '@/assets/newCourseList.json';
import courseDetail from '@/assets/newCourseDetail.json';

const fullList = ref(courses.course);
const Detail = ref(courseDetail);

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

async function search() {
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

search();

const tableItems = ref({});
const tabs = ref([]);
const restrictions = ref({});
const activeTab = ref('');
function getDetail(courseCode) {
    // 取出 courseDetail[courseCode].category 的键值
    // category 是一个 Object[]，所以用 map 取出第一个键值
    tabs.value = courseDetail[courseCode].category.map(obj => Object.keys(obj)[0]);
    activeTab.value = tabs.value[0];
    for (let tab of tabs.value) {
        tableItems.value[tab] = courseDetail[courseCode].activities.filter(activity => {
            return activity.category === tab && activity.day !== 'X';
        });
        restrictions.value[tab] = courseDetail[courseCode].category[tab];
        if (restrictions.value[tab] === undefined) {
            restrictions.value[tab] = ['None'];
        } else {
            restrictions.value[tab] = restrictions.value[tab][0];
        }
    }
    for (let tab of tabs.value) {
        for (let activity of tableItems.value[tab]) {
            activity.dayTime = char2Day[activity.day] + ' ' + activity.startTime + '-' + activity.endTime;
            activity.buildingRoom = activity.bldg + ' ' + activity.room;
            activity.webEnabled = activity.web ? 'Y' : 'N';
        }
    }
    // console.log(tableItems.value);
}

function handleTabClick(tab, event) {
    activeTab.value = tab.props.name;
}

function handleCatalogClick(courseCode) {
    window.open(`http://www.cityu.edu.hk/catalogue/ug/current/course/${courseCode}.htm`, '_blank');
}

// getDetail("CS1302");
</script>

<template>
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
        <el-table :data="currentList" border stripe>
            <el-table-column class="table-medium" prop="courseCode" label="Course Code" min-width="2"></el-table-column>
            <el-table-column class="table-long" prop="courseTitle" label="Course Title" min-width="4"></el-table-column>
            <el-table-column class="table-short" prop="creditUnits" label="Credit Units" min-width="1"></el-table-column>
            <el-table-column class="table-short" prop="webEnabled" label="Web Enabled" min-width="1"></el-table-column>
            <el-table-column class="table-medium" prop="details" label="Details" min-width="2">
                <template #default="{row}">
                    <el-button type="text" @click="handleCatalogClick(row.courseCode)">Catalog</el-button>
                    <el-button type="text" @click="getDetail(row.courseCode)">Details</el-button>
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

    <!--
    展示某门课的详细信息
    上方显示 tabs，分类为所有的 Category
    下方表格显示每个 Section 的信息，包括 CRN, Section, Day / Time, Building / Room, 和加入到 Timetable 的按钮
    -->
    <div>
        <div class="tab-container">
            <el-tabs value="activeTab" @tab-click="handleTabClick" :type="'card'">
                <el-tab-pane v-for="item in tabs" :key="item" :label="item" :name="item"></el-tab-pane>
            </el-tabs>
            <p>Restrictions:</p>
            <ul>
                <li v-for="item in restrictions[activeTab]" :key="item">{{item}}</li>
            </ul>
            <div class="table-container">
                <el-table :data="tableItems[activeTab]" border stripe>
                    <el-table-column class="table-short" prop="crn" label="CRN" min-width="1"></el-table-column>
                    <el-table-column class="table-short" prop="section" label="Section" min-width="1"></el-table-column>
                    <el-table-column class="table-medium" prop="dayTime" label="Time" min-width="2"></el-table-column>
                    <el-table-column class="table-medium" prop="buildingRoom" label="Venue" min-width="2"></el-table-column>
                    <el-table-column class="table-short" prop="webEnabled" label="Web" min-width="1"></el-table-column>
                    <el-table-column clash="table-short" prop="status" label="Status" min-width="1">
                        <template #default="{row}">
                            <p>OK</p> <!-- TODO: 判断是否存在冲突 -->
                        </template>
                    </el-table-column>
                    <el-table-column class="table-medium" prop="add" label="Add" min-width="2">
                        <template #default="{row}">
                            <el-button type="text" @click="addToTimetable(row)">Add</el-button>
                        </template>
                    </el-table-column>
                </el-table>
            </div>
        </div>
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
</style>