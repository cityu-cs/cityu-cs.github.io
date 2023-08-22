<script setup>
import CourseTable from "@/components/CourseTable.vue";
</script>

<template>
    <header>
        <div>
            <h1>CityU Schedule Helper</h1>
            <p>
                For Semester A, 2023/24
            </p>
        </div>
    </header>
    <main>
        <CourseTable />
    </main>
</template>

<style scoped>
header {
    line-height: 1.5;
    text-align: center;
}
header > div {
    max-width: 800px;
    margin: 0 auto 20px;
}
</style>
