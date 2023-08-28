<script setup>
import CourseList from "@/components/CourseList.vue";
import Timetable from "@/components/Timetable.vue";
</script>

<template>
    <header>
        <div>
            <h1>CityU Schedule Helper</h1>
            <p>
                SEMESTER A, 2023/24
            </p>
        </div>
    </header>
    <main>
        <CourseList />
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
