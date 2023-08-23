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

// 当没有搜索条件时，显示全部课程

const queryText = ref('');
const querySubject = ref('');
const queryLevel = ref('');
// const resultList = ref(fullList.value);
const resultList = ref([]);

async function search() {
    const query = queryText.value.toLowerCase();
    const subject = querySubject.value;
    const level = queryLevel.value;

    if (!query && !subject && !level) {
        resultList.value = fullList.value;
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
}
</script>

<template>
    <div class="Search">
        <el-input placeholder="Course Code or Title" v-model="queryText" clearable></el-input>
        <el-select v-model="querySubject" placeholder="Subject" clearable>
            <el-option
                    v-for="item in subjectList"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
            </el-option>
        </el-select>
        <el-select v-model="queryLevel" placeholder="Level" clearable>
            <el-option
                    v-for="item in levelList"
                    :key="item.value"
                    :label="item.label"
                    :value="item.value">
            </el-option>
        </el-select>
        <el-button type="primary" @click="search">Search</el-button>
    </div>

    <div class="Table">
        <el-table :data="resultList" border stripe>
            <el-table-column prop="courseCode" label="Course Code" width="220px"></el-table-column>
            <el-table-column prop="courseTitle" label="Course Title" width="660px"></el-table-column>
            <el-table-column prop="creditUnits" label="Credit Units" width="110px"></el-table-column>
            <el-table-column prop="webEnabled" label="Web Enabled" width="110px"></el-table-column>
            <el-table-column prop="details" label="Details">
                <template #default="{row}">
                    <el-button type="text" @click="window.open(row.details, '_blank')">Details</el-button>
                </template>
            </el-table-column>
        </el-table>
    </div>
</template>

<style scoped>
.Search {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
}
</style>