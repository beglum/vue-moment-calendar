<script>
import Vue from 'vue';
import VueMomentCalendar from '@/vue-moment-calendar.vue';

export default Vue.extend({
  name: 'ServeDev',
  data: () => ({
    events: [
      {
        id: 1,
        name: 'Коммуникационный дизайн среды',
        date: '05.10.21',
        type: 'red',
      },
      {
        id: 7,
        name: 'Коммуникационный дизайн среды',
        date: '03.11.21',
        type: 'red',
      },
      {
        id: 2,
        name: 'Коммуникационный дизайн среды',
        date: '05.11.2021 15:00',
        type: 'green',
      },
      {
        id: 3,
        name: 'Коммуникационный дизайн среды',
        date: '05.11.2021 16:00',
        type: 'orange',
      },
      {
        id: 4,
        name: 'Коммуникационный дизайн среды',
        date: '04.12.21 18:00',
        type: 'orange',
      },
      {
        id: 5,
        name: 'Коммуникационный дизайн среды',
        date: '04.09.21 18:00',
        type: 'orange',
      },
    ],
  }),
  components: {
    VueMomentCalendar,
  },
});
</script>

<template>
  <div id="app">
    <vue-moment-calendar :events="events" />
  </div>
</template>
