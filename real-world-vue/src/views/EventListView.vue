<script setup lang="ts">
import EventCard from '../components/EventCard.vue'
import type { EventItem } from '@/type'
import { ref, watchEffect, type Ref, computed } from 'vue'
import EventService from '@/services/EventService'
import type { AxiosResponse } from 'axios'
import { useRouter } from 'vue-router'


const events: Ref<EventItem[]> = ref([])
const totalEvent = ref<number>(0)
const props = defineProps({
  page: {
    type: Number,
    required: true
  },
  limit: {
    type: Number,
    required: true
  }
})

EventService.getEvent(props.limit, props.page).then((response: AxiosResponse<EventItem[]>) => {
  events.value = response.data
  totalEvent.value = response.headers['x-total-count']
})

watchEffect(() => {
  EventService.getEvent(props.limit, props.page).then((response: AxiosResponse<EventItem[]>) => {
  events.value = response.data
  totalEvent.value = response.headers['x-total-count']
  })
})

const router = useRouter()

const limit = ref(props.limit)

const increaseLimit = () => {
  limit.value++
  router.push({ name: 'event-list', query: { page: props.page, limit: limit.value } })
}

const decreaseLimit = () => {
  if (limit.value > 1) {
    limit.value--
    router.push({ name: 'event-list', query: { page: props.page, limit: limit.value } })
  }
}

const hasNextPage = computed(() => {
  //first calculate the total page
  const totalPages = Math.ceil(totalEvent.value / 2)
  return props.page.valueOf() < totalPages
})
</script>

<template>
  <h1>
    Events For Good <button @click="increaseLimit">plus</button>
  
  <button @click="decreaseLimit">minus</button>
    {{ limit }}
  </h1>
  <main class="events">
    <EventCard v-for="event in events" :key="event.id" :event="event"></EventCard>
    <div class="pagination">
      <RouterLink
        :to="{ name: 'event-list', query: { page: page - 1 } }"
        rel="prev"
        v-if="page != 1"
      >
        Prev Page
      </RouterLink>
      <RouterLink
        :to="{ name: 'event-list', query: { page: page + 1 } }"
        rel="next"
        v-if="hasNextPage"
      >
        Next Page
      </RouterLink>
    </div>
  </main>
</template>

<style scoped>
.events {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.events2 {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: right;
}
 h1 button {
  background-color: #2c3e50;
  border-radius: 10px;
  color: white;
  font-size: 20px;
}

.pagination {
  display: flex;
  width: 290px;
}

.pagination a {
  flex: 1;
  text-decoration: none;
  color: #2c3e50;
}

#page-prev {
  text-align: left;
}

#page-next {
  text-align: right;
}
</style>
