<script setup>
import { ref } from 'vue';
import courses from '@/assets/courses.json';

const fullList = ref([]);
for (const subject in courses) {
    for (const course of courses[subject]) {
        fullList.value.push(course);
    }
}

const subjectList = ref([]);
for (const subject in courses) {
    subjectList.value.push({
        value: subject,
        label: subject
    });
}

const levelList = ref([
    {value: 'A', label: 'Associate Degree'},
    {value: 'B', label: 'Bachelor\'s Degree'},
    {value: 'P', label: 'Postgraduate Degree'},
    {value: 'D', label: 'Professional Doctorate'},
    {value: 'R', label: 'Research Degree'}
]);

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
                    <el-button type="text" @click="window.open(row.details, '_blank')">Details</el-button>
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
</style>