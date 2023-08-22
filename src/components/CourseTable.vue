<script setup>
import { ref } from 'vue'
import courses from '@/assets/courses.json'
const tableData = ref([])
for (const subject in courses) {
    for (const course of courses[subject]) {
        tableData.value.push(course)
    }
}
</script>
<template>
    <el-table :data="tableData" border>
        <el-table-column prop="courseCode" label="Course Code"></el-table-column>
        <el-table-column prop="courseTitle" label="Course Title"></el-table-column>
        <el-table-column prop="creditUnits" label="Credit Units"></el-table-column>
        <el-table-column prop="webEnabled" label="Web Enabled"></el-table-column>
    </el-table>
</template>

<style scoped>
el-table {
    width: 100%;
}
</style>