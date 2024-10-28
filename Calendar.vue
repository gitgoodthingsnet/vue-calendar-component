<template>
  <div class="container vue-calendar">
    <div class="row">
      <div class="col">
        <!-- Calendar Header -->
        <div
          v-if="currentView === 'month'"
          class="btn-group btn-group mb-3"
          role="group"
          aria-label="..."
        >
          <button class="btn btn-primary" @click="prevMonth"><</button>
          <button class="btn btn-outline-primary">
            {{ monthName }} {{ currentYear }}
          </button>
          <button class="btn btn-primary" @click="nextMonth">></button>
        </div>
        <div
          class="float-end btn-group btn-group mb-3"
          role="group"
          aria-label="..."
        >
          <button
            class="btn"
            :class="
              currentView === 'month' ? 'btn-primary' : 'btn-outline-primary'
            "
            @click="switchView('month')"
          >
            Month
          </button>

          <!-- Button to switch to week view -->
          <button
            class="btn"
            :class="
              currentView === 'week' ? 'btn-primary' : 'btn-outline-primary'
            "
            @click="switchView('week')"
          >
            Week
          </button>

          <!-- Button to switch to day view -->
          <button
            class="btn"
            :class="
              currentView === 'day' ? 'btn-primary' : 'btn-outline-primary'
            "
            @click="switchView('day')"
          >
            Day
          </button>
        </div>
        <!-- Week navigation buttons -->
        <div v-if="currentView === 'week'" class="btn-group">
          <button class="btn btn-primary" @click="prevWeek"><</button>
          <button class="btn btn-primary" @click="nextWeek">></button>
        </div>
      </div>
    </div>

    <!-- Month view -->
    <div v-if="currentView === 'month'">
      <div class="row text-center font-weight-bold">
        <div class="col" v-for="(day, index) in daysOfWeek" :key="index">
          {{ day }}
        </div>
      </div>

      <!-- Days in the Month (7 days per row) -->
      <div class="row">
        <div v-for="(week, index) in weeksInMonth" :key="index" class="d-flex">
          <div
            class="col border p-2 d-flex flex-column justify-content-between day-cell"
            v-for="day in week"
            :key="day.date"
            :class="{ 'bg-white': true, 'text-muted': !day.isCurrentMonth }"
          >
            <!-- Day number -->
            <div>
              <span>{{ day.date.getDate() }}</span>
            </div>
            <!-- Events for the day -->
            <div v-if="day.events.length" class="mt-2">
              <div
                v-for="(event, index) in sortEventsByStartTime(day.events)"
                :key="index"
                :class="getEventClass(day.date, event)"
                class="event-badge badge bg-primary mb-1"
                @click="onEventClick(event.id)"
              >
                <div
                  v-if="!event.allday && event.start"
                  class="event-badge-dot"
                ></div>
                <div
                  v-if="!event.allday && event.start"
                  class="event-badge-time"
                >
                  {{ new Date(event.start).getHours() }}:{{
                    new Date(event.start)
                      .getMinutes()
                      .toString()
                      .padStart(2, "0")
                  }}
                </div>
                <div class="event-badge-title">{{ event.name }}</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Week view -->
    <div v-else-if="currentView === 'week'">
      <!-- Week days headers -->
      <div class="row text-center font-weight-bold">
        <div class="col-1"></div>

        <div class="col" v-for="day in currentWeek" :key="day.date">
          {{ getDayName(day.date) }} {{ day.date.getDate() }}
        </div>
      </div>
      <div class="row text-center font-weight-bold">
        <div class="col-1 border-bottom">All Day</div>
        <div
          v-for="day in currentWeek"
          :key="day.date"
          class="col p-0 border-start allday border-bottom"
        >
          <!-- Render events this day -->
          <div
            v-for="(event, index) in getEventsInHour(day, null, true)"
            :key="index"
          >
            <div
              class="event-badge-allday ms-0 badge bg-primary"
              @click="onEventClick(event.id)"
            >
              {{ event.name }}
            </div>
          </div>
        </div>
      </div>

      <!-- Time slots for each day of the week -->
      <div class="row">
        <!-- Time column (00:00 to 23:00) -->
        <div class="col-1 p-0">
          <div v-for="hour in hours" :key="hour" class="hour-slot">
            {{ hour }}:00
          </div>
        </div>

        <!-- Days (Sunday to Saturday) in columns with hourly events -->
        <div
          v-for="day in currentWeek"
          :key="day.date"
          class="col p-0 border-start"
        >
          <div v-for="hour in hours" :key="hour" class="hour-slot">
            <!-- Render events for this hour -->
            <div
              v-for="(event, index) in getEventsInHour(day, hour, false)"
              :key="index"
            >
              <template
                v-if="
                  event &&
                  event.start &&
                  !isNaN(new Date(event.start).getTime()) &&
                  new Date(event.start).getHours() === hour
                "
              >
                <!-- Calculate overlapping events and their positions -->
                <div
                  :style="
                    getEventStyle(
                      event,
                      getOverlappingEvents(day, event).length,
                      getOverlappingEvents(day, event).indexOf(event)
                    )
                  "
                  class="event-badge-day badge bg-primary pointer"
                  @click="onEventClick(event.id)"
                >
                  <div class="text-start event-badge-title">
                    {{ event.name }}
                  </div>
                  <div class="text-start mt-2">
                    {{ new Date(event.start).getHours() }}:{{
                      new Date(event.start)
                        .getMinutes()
                        .toString()
                        .padStart(2, "0")
                    }}
                    - {{ new Date(event.end).getHours() }}:{{
                      new Date(event.end)
                        .getMinutes()
                        .toString()
                        .padStart(2, "0")
                    }}
                  </div>
                </div>
              </template>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

const props = defineProps({
  events: {
    type: Array,
    required: true,
  },
  eventClick: {
    type: Function,
    default: () => {},
  },
});

const currentView = ref("month");
const currentWeekIndex = ref(0);
const currentMonth = ref(new Date().getMonth());
const currentYear = ref(new Date().getFullYear());
const daysOfWeek = computed(() => [
  "Sun",
  "Mon",
  "Tue",
  "Wed",
  "Thu",
  "Fri",
  "Sat",
]);

const onEventClick = (eventId) => {
  props.eventClick(eventId);
};

const getEventStyle = (
  event,
  overlappingCount = 1,
  positionIndex = 0,
  hourHeightPx = 60
) => {
  if (!event || !event.start || !event.end) {
    return {};
  }

  const eventStartDate = new Date(event.start);
  const eventEndDate = new Date(event.end);

  if (isNaN(eventStartDate.getTime()) || isNaN(eventEndDate.getTime())) {
    return {};
  }

  // Define the visible hours range (e.g., from 08:00 to 18:00)
  const visibleStartHour = 0;
  const visibleEndHour = 24;

  // Calculate start and end hours
  const startHour =
    eventStartDate.getHours() + eventStartDate.getMinutes() / 60;
  const endHour = eventEndDate.getHours() + eventEndDate.getMinutes() / 60;

  // If the event is outside the visible hours, return an empty style
  if (startHour < visibleStartHour || endHour > visibleEndHour) {
    return {};
  }

  // Calculate the top and height in pixels
  const topPx = (eventStartDate.getMinutes() / 60) * hourHeightPx;
  const durationHours = endHour - startHour;
  const heightPx = durationHours * hourHeightPx;

  // Calculate the width and left position for overlapping events
  const eventWidth = 100 / overlappingCount; // Divide by the number of overlapping events
  const leftPx = positionIndex * eventWidth;

  return {
    top: `${topPx}px`, // Top position in pixels
    left: `${leftPx}%`, // Left position in percentage
    height: `${heightPx}px`, // Height in pixels
    width: `${eventWidth}%`, // Width in percentage based on overlapping
    position: "absolute",
  };
};

const getOverlappingEvents = (day, event) => {
  // Filter to find overlapping events, but exclude all-day events
  const overlappingEvents = day.events.filter((otherEvent) => {
    if (otherEvent.allday) {
      return false; // Exclude all-day events from the overlapping calculation
    }

    const otherStart = new Date(otherEvent.start).getTime();
    const otherEnd = new Date(otherEvent.end).getTime();
    const eventStart = new Date(event.start).getTime();
    const eventEnd = new Date(event.end).getTime();

    // Check if the events overlap
    return eventStart < otherEnd && otherStart < eventEnd;
  });

  return overlappingEvents;
};

const getDayName = (date) => {
  return date.toLocaleString("default", { weekday: "short" }); // Outputs: "Sun", "Mon", etc.
};

const monthName = computed(() => {
  return new Date(currentYear.value, currentMonth.value).toLocaleString(
    "default",
    { month: "long" }
  );
});

// Helper function to strip time from date and compare only by year, month, and day
const isSameDay = (date1, date2) => {
  return (
    date1.getFullYear() === date2.getFullYear() &&
    date1.getMonth() === date2.getMonth() &&
    date1.getDate() === date2.getDate()
  );
};

// Compute the days in the month and arrange them in weeks
const weeksInMonth = computed(() => {
  const days = [];
  const startOfMonth = new Date(currentYear.value, currentMonth.value, 1);
  const endOfMonth = new Date(currentYear.value, currentMonth.value + 1, 0);

  // Get the day of the week for the start of the month (0 = Sunday, 6 = Saturday)
  const leadingDays = startOfMonth.getDay();

  // Calculate the days from the previous month
  for (let i = 0; i < leadingDays; i++) {
    days.push({
      date: new Date(
        currentYear.value,
        currentMonth.value,
        i - leadingDays + 1
      ),
      isCurrentMonth: false,
      events: [], // Ensure events is an empty array
    });
  }

  // Calculate the days in the current month
  for (let i = 1; i <= endOfMonth.getDate(); i++) {
    const dayDate = new Date(currentYear.value, currentMonth.value, i);
    const dayEvents = props.events.filter((event) => {
      const eventStartDate = new Date(event.start);
      const eventEndDate = new Date(event.end);
      return (
        isSameDay(dayDate, eventStartDate) ||
        (eventStartDate <= dayDate && dayDate <= eventEndDate)
      );
    });
    days.push({
      date: dayDate,
      isCurrentMonth: true,
      events: dayEvents || [], // Populate events correctly
    });
  }

  // Add days from the next month to fill the week
  const trailingDays = 6 - endOfMonth.getDay();
  for (let i = 1; i <= trailingDays; i++) {
    days.push({
      date: new Date(currentYear.value, currentMonth.value + 1, i),
      isCurrentMonth: false,
      events: [], // Ensure events is an empty array
    });
  }

  // Split the days into weeks
  const weeks = [];
  for (let i = 0; i < days.length; i += 7) {
    weeks.push(days.slice(i, i + 7));
  }

  return weeks;
});

// Get the current week based on the currentWeekIndex
const currentWeek = computed(() => {
  return weeksInMonth.value[currentWeekIndex.value] || [];
});

const prevMonth = () => {
  if (currentMonth.value === 0) {
    currentMonth.value = 11;
    currentYear.value--;
  } else {
    currentMonth.value--;
  }
};

const nextMonth = () => {
  if (currentMonth.value === 11) {
    currentMonth.value = 0;
    currentYear.value++;
  } else {
    currentMonth.value++;
  }
};

const prevWeek = () => {
  if (currentWeekIndex.value > 0) {
    currentWeekIndex.value--;
  }
};

const nextWeek = () => {
  if (currentWeekIndex.value < weeksInMonth.value.length - 1) {
    currentWeekIndex.value++;
  }
};

// Array of hours (00:00 to 23:00)
const hours = Array.from({ length: 24 }, (_, i) => i);

// Function to get events for a specific hour in a specific day
const getEventsInHour = (day, hour, getalldayevent) => {
  return day.events.filter((event) => {
    // Set allday to false if it's undefined
    event.allday = event.allday ?? false;

    // If we are fetching all-day events
    if (getalldayevent) {
      return event.allday;
    }

    // If we are fetching hourly events, exclude all-day events
    if (!event.allday) {
      const eventStartDate = new Date(event.start);
      const eventEndDate = new Date(event.end);

      // Check if the event spans the current hour
      return (
        eventStartDate.getHours() <= hour && eventEndDate.getHours() >= hour
      );
    }

    return false;
  });
};

// Function to get the class for the event based on its position
const getEventClass = (dayDate, event) => {
  let _class = "";
  const eventStartDate = new Date(event.start);
  const eventEndDate = new Date(event.end);
  if (isSameDay(dayDate, eventStartDate)) {
    if (isSameDay(dayDate, eventEndDate)) {
      _class = "event-start event-end";
    } else {
      _class = "event-start";
    }
  } else if (isSameDay(dayDate, eventEndDate)) {
    _class = "event-end";
  } else {
    _class = "event-middle";
  }
  switch (event.type) {
    case "danger":
      return _class + " bg-danger";
    case "success":
      return _class + " bg-success";
    case "info":
      return _class + " bg-info";
    case "primary":
      return _class + " bg-primary";
    case "secondary":
      return _class + " bg-secondary";
    case "warning":
      return _class + " bg-warning";
    case "dark":
      return _class + " bg-dark";
    case "light":
      return _class + " bg-light";
    default:
      return _class + " bg-warning";
  }
};

const sortEventsByStartTime = (events) => {
  return events.slice().sort((a, b) => new Date(a.start) - new Date(b.start));
};

const getCurrentWeekIndex = () => {
  const today = new Date();
  const currentMonthWeeks = weeksInMonth.value;

  for (let i = 0; i < currentMonthWeeks.length; i++) {
    const week = currentMonthWeeks[i];
    const found = week.find((day) => isSameDay(day.date, today));
    if (found) {
      return i;
    }
  }
  return 0; // Default to the first week if the current week isn't found
};

const switchView = (view) => {
  currentView.value = view;

  if (view === "week") {
    currentWeekIndex.value = getCurrentWeekIndex(); // Set to the current week
  } else {
    currentWeekIndex.value = 0; // Reset week index when switching to month view
  }
};
</script>

<style scoped>
.day-cell {
  min-height: 100px;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  width: 14.28%;
  max-width: 14.28%;
  box-sizing: border-box;
}

.bg-light {
  background-color: #f8f9fa !important;
}

.vue-calendar .event-badge,
.vue-calendar .event-badge-allday {
  margin-left: -5px;
  padding: 8px 10px;
  border-width: 0 0 0 4px;
  margin-bottom: 1px;
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
  display: flex;
}

.vue-calendar .event-start {
  border-top-left-radius: 5px;
  border-bottom-left-radius: 5px;
  border-top-right-radius: 0px;
  border-bottom-right-radius: 0px;
  margin-right: -10px;
  border-width: 0 0 0 6px;
  border-left: solid;
}

.vue-calendar .event-middle {
  border-top-left-radius: 0x !important;
  border-bottom-left-radius: 0px;
  border-top-right-radius: 0px;
  border-bottom-right-radius: 0px;
  margin-right: -10px;
  margin-left: -10px;
}

.vue-calendar .event-end {
  border-top-left-radius: 0x !important;
  border-bottom-left-radius: 0px;
  border-top-right-radius: 20px;
  border-bottom-right-radius: 20px;
  margin-left: -10px;
}

.vue-calendar .hour-slot {
  height: 60px;
  border-bottom: 1px solid #e9ecef;
  align-items: center;
  justify-content: center;
  position: relative;
}

.vue-calendar .badge {
  border-width: 0 0 0 4px;
}

.vue-calendar .badge.bg-primary {
  color: var(--bs-primary);
  background-color: var(--bs-primary-bg-subtle) !important;
  border-color: var(--bs-primary);
}

.vue-calendar .badge.bg-secondary {
  color: var(--bs-secondary);
  background-color: var(--bs-secondary-bg-subtle) !important;
  border-color: var(--bs-secondary);
}

.vue-calendar .badge.bg-success {
  color: var(--bs-success);
  background-color: var(--bs-success-bg-subtle) !important;
  border-color: var(--bs-success);
}

.vue-calendar .badge.bg-danger {
  color: var(--bs-danger);
  background-color: var(--bs-danger-bg-subtle) !important;
  border-color: var(--bs-danger);
}

.vue-calendar .badge.bg-info {
  color: var(--bs-info);
  background-color: var(--bs-info-bg-subtle) !important;
  border-color: var(--bs-info);
}

.vue-calendar .badge.bg-warning {
  color: var(--bs-warning);
  background-color: var(--bs-warning-bg-subtle) !important;
  border-color: var(--bs-warning);
}

.vue-calendar .event-badge-day,
.event-badge-allday {
  border-left: solid 4px;
  cursor: pointer;
}

.vue-calendar .event-badge-dot {
  border: 4px solid;
  border-radius: 4px;
  box-sizing: content-box;
  height: 0px;
  width: 0px;
  top: 4px;
  position: relative;
  margin: 0 5px 0 0;
}

.vue-calendar .event-badge-time {
  margin-right: 5px;
}

.vue-calendar .event-badge-title {
  white-space: nowrap;
  overflow: hidden;
}
</style>
