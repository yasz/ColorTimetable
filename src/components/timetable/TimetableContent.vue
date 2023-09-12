<script setup lang="ts">
import { findIndex } from 'lodash-es'
import TimetableAction from './TimetableAction.vue'
import TimetableHeader from './TimetableHeader.vue'
import type { CourseModel } from '~/stores/course'
import { courseTimeList } from '~/stores/course'

withDefaults(
  defineProps<{ showCourseAction: boolean }>(), {
  showCourseAction: false,
},
)
const divideIndex = ref(-1)
courseTimeList.some((courseTime, index) => {
  if (courseTime.s > '12') {
    divideIndex.value = index;
    return true; // 中止遍历
  }
  return false; // 继续遍历
});
courseTimeList.splice(divideIndex.value, 0, {})
console.log('【】:', courseTimeList);
const emit = defineEmits(['courseItemClick'])

const { customBarHeight } = storeToRefs(useAppStore())
const { weekNum, weekCourseList, currentWeekIndex, originalWeekIndex } = storeToRefs(useCourseStore())
const { hasConflictCourseByMap, setCurrentWeekIndex } = useCourseStore()

// delete a course when course at the same time
const deleteWeekCourse = computed(() => {
  const _weekCourse = [...weekCourseList.value]
  _weekCourse.forEach((e, i) => {
    if (e.start > divideIndex.value) {
      e.start = e.start + 1
    }
  })
  for (let i = 1; i < _weekCourse.length; i++) {
    const { start, week } = _weekCourse[i]
    const { start: prevStart, week: prevWeek } = _weekCourse[i - 1]
    if (start === prevStart && week === prevWeek) {
      _weekCourse.splice(i, 1)
      i--
    }
  }
  return _weekCourse
})

const startX = ref(0)
const startY = ref(0)
const towardsX = ref(0)
const towardsY = ref(0)


function resetTouchStatus() {
  startX.value = 0
  startY.value = 0
  towardsX.value = 0
  towardsY.value = 0
}

function handleTouchStart(e: TouchEvent) {
  startX.value = e.touches[0].clientX
  startY.value = e.touches[0].clientY
}

function handleTouchMove(e: TouchEvent) {
  towardsX.value = e.touches[0].clientX - startX.value
  towardsY.value = e.touches[0].clientY - startY.value
}

function handleTouchEnd() {
  let currentWeekIndexTemp = currentWeekIndex.value
  if (towardsX.value === 0 || Math.abs(towardsY.value) > 50)
    return
  if (towardsX.value > 50) {
    if (currentWeekIndexTemp === 0)
      return
    currentWeekIndexTemp--
  }
  else if (towardsX.value < -50) {
    if (currentWeekIndexTemp === weekNum.value - 1)
      return
    currentWeekIndexTemp++
  }
  setCurrentWeekIndex(currentWeekIndexTemp)
  resetTouchStatus()
}

/**
 * get course position
 * @param item course item
 * @returns css style
 */
function getCoursePosition(item: CourseModel) {
  return {
    'grid-row': `${item.start} / ${item.start + item.duration}`,
    'grid-column': `${item.week + 1} / ${item.week + 1 + 1}`,
  }
}
</script>

<template>
  <div class="overflow-y-scroll relative bg-base" :style="{ height: `calc(100vh - ${customBarHeight}px)` }">
    <div class="w-full top-0 z-10 fixed bg-base" :style="{ 'padding-top': `${customBarHeight}px` }">
      <TimetableAction :show-course-action="showCourseAction" />
      <TimetableHeader />
    </div>

    <div class="min-h-max pb-safe p-1 transition-all z-20 duration-300 bg-base"
      grid="~ flow-col rows-10 cols-[0.7fr_repeat(7,1fr)] gap-1" :class="showCourseAction ? 'pt-31' : 'pt-11'"
      @touchstart="handleTouchStart" @touchmove="handleTouchMove" @touchend="handleTouchEnd">
      <template v-for="(courseTime, courseIndex) in courseTimeList" :key="courseIndex">
        <div class="text-sm min-h-18" flex="~ col" justify-evenly items-center>


          <div v-if="courseIndex != divideIndex" class="font-medium">
            {{ courseIndex > divideIndex ? courseIndex : courseIndex + 1 }}
          </div>

          <div class="px-0.5 text-8px">
            {{ courseTime.s }}<br>{{ courseTime.e }}
          </div>
        </div>
      </template>

      <template v-for="(courseItem, _courseIndex) of deleteWeekCourse" :key="_courseIndex">

        <div class="rounded-lg p-0.5 relative dark:bg-op40" b="white 2 !op-50"
          :style="[getCoursePosition(courseItem), `background-color:${hasConflictCourseByMap(courseItem)[0].color}`]"
          @click="emit('courseItemClick', courseItem)">

          <div class="h-full w-full" text="center white xs" flex="~ col" justify-around items-center>
            <div class="font-medium break-all">
              {{ hasConflictCourseByMap(courseItem)[0].title }}
            </div>
            <div v-if="hasConflictCourseByMap(courseItem)[0].location != '莘塍'" class="break-all">
              {{ hasConflictCourseByMap(courseItem)[0].location }}
            </div>
            <div class="text-1">
              {{ hasConflictCourseByMap(courseItem)[0].teacher }}
            </div>
            <div v-if="hasConflictCourseByMap(courseItem).length > 1"
              class="rounded h-1 top-1 left-1 right-1 absolute bg-white/80" />
          </div>

        </div>
      </template>
    </div>
    <div class="bg-primary fixed top-40% z-30 rounded-l-full transition-all duration-300" text="white sm" p="l-4 y-2 r-2"
      :class="originalWeekIndex !== currentWeekIndex ? 'right-0' : '-right-full'"
      @click="setCurrentWeekIndex(originalWeekIndex)">
      返回本周
    </div>
  </div>
</template>
<style></style>