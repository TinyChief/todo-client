<template>
  <div :class="['wrapper', this.$store.getters.menuStatus ? 'menu-opened' : 'menu-closed']">
    <Toolbar leftIcon="fal fa-bars" title="" @left-icon-clicked="leftAction">
    </Toolbar>
    <vue-calendar @show-all="showAll" @day-clicked="dayClicked" @event-clicked="eventClicked" @month-changed="monthChanged">
    </vue-calendar>
    <div class="blocks">
      <div v-for="(hero, index) in tasksDisplay" v-bind:key="index" class="block-item">
        <!-- <img :src="require('../assets/images/people.png')" :alt="hero.type"> -->
        <Icon :iconClass="hero.icon"></Icon>
        <div class="block-item__content">
          <p class="block-item__title">
            {{ hero.title }}
            <span>{{hero.time}}</span>
          </p>
          <p class="block-item__type">{{hero.type}}</p>
        </div>
      </div>
    </div>
    <add-task></add-task>
  </div>
</template>

<script>
import Toolbar from '@/components/reusable/Toolbar.vue';
import AddTask from '@/components/reusable/AddTask.vue';

export default {
  name: 'Home',
  components: {
    Toolbar,
    AddTask,
  },
  data() {
    return {
      toggleMenu: false,
      tasksDisplay: [],
      tasksFiltered: [],
      tasks: [],
    };
  },
  methods: {
    leftAction: function() {
      this.$store.dispatch('toggleMenu');
    },
    showAll: function(events) {
      // Do something...
    },
    dayClicked: function(day) {
      if (day.date) {
        day = day.date
      }
      // When day pressed display tasks belonging to this day
      this.tasksDisplay = this.tasksFiltered.filter(el => {
        if (
          el.date.getMonth() === day.getMonth() &&
          el.date.getDate() === day.getDate()
        ) {
          return true
        }
      });
    },
    eventClicked: function(event) {
      // Do something...
    },
    monthChanged: function(start, end) {
      handleHighlights(this.tasksFiltered, 'remove');
      const month = end.getMonth();
      const year = end.getFullYear();
      this.tasksFiltered = filterTasksByMonthAndYear(this.tasks, year, month);
      this.$nextTick(() => {
        handleHighlights(this.tasksFiltered, 'add');
      });
    }
  },
  mounted: function() {
    const oldNext = document.querySelector('.next-month');
    oldNext.innerHTML = '<i class="fal fa-angle-right"> </i>';
    const oldPrev = document.querySelector('.prev-month');
    oldPrev.innerHTML = '<i class="fal fa-angle-left"> </i>';
    this.$nextTick(() => {
      this.dayClicked(this.$store.getters.getTodayDate)
    });
  },
  beforeMount: function() {
    this.tasks = this.$store.getters.userTasks
  },
};

function filterTasksByMonthAndYear(tasks, year, month) {
  return tasks.filter((el, i) => {
    el.date = new Date(el.date);
    const elMonth = el.date.getMonth();
    const elYear = el.date.getFullYear();
    if (elMonth == month && elYear == year) {
      return true;
    } else {
      return false;
    }
  });
}

function handleHighlights(tasks, action) {
  const days = document.querySelectorAll('.week-day');
  tasks.forEach(el => {
    const taskDay = el.date.getDate();
    const indexOfDay = Array.from(days).findIndex((el, i) => {
      if (
        !Array.from(days[i].classList).includes('not-current') &&
        el.innerText.trim() == taskDay
      ) {
        return true;
      }
    });
    days[indexOfDay].classList[action]('highlighted');
  });
}
</script>

<style scoped lang="scss">
i {
  font-size: 36px;
}
.wrapper {
  position: relative;
}

.blocks {
  overflow-y: scroll;
  max-height: 250px;
}
</style>
