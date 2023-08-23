<script setup>
// generate 10 human-readable colors
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

const char2day = {
    M: 'Monday',
    T: 'Tuesday',
    W: 'Wednesday',
    R: 'Thursday',
    F: 'Friday',
    S: 'Saturday',
    U: 'Sunday',
    X: 'Unknown'
};
</script>

<template>
    <!-- 以表格形式呈现课程列表 -->
    <!-- Monday ~ Sunday, 09:00 - 23:00 -->
    <div class="timetable">
        <table>
            <thead>
                <tr>
                    <th></th>
                    <th v-for="day in 7" :key="day">{{ day }}</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="hour in 15" :key="hour">
                    <td>{{ hour + 8 }}:00</td>
                    <td v-for="day in 7" :key="day" :style="{ backgroundColor: selectColor(day2number[day] * hour) }">
<!--                        <div v-for="course in timetable[day][hour]" :key="course.courseCode">-->
<!--                            {{ course.courseCode }}-->
<!--                        </div>-->
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</template>

<style scoped>

</style>