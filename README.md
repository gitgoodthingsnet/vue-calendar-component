Here’s a sample `README.md` file for your calendar component:

```md
# Vue Calendar Component

A flexible and customizable calendar component built using **Vue.js 3**, with support for **month, week, and day views**. This component supports various event types, including **all-day events** and **time-specific events**.

## Features

- **Month View**: Displays a full month, with day cells showing event badges.
- **Week View**: Displays the current week with time slots for events.
- **Day View**: Focus on a single day with events displayed per time slot.
- **Event Types**: Support for all-day events and time-specific events.
- **Responsive**: Adjusts the layout based on the screen size.
- **Event Click Callback**: Handle event clicks using a callback function.

## Installation

1. Clone or download the repository from GitHub.
2. Install dependencies:

   ```bash
   npm install
   ```

3. Import the component into your project:

   ```js
   import Calendar from './components/Calendar.vue';
   ```

4. Use the component in your Vue project:

   ```vue
   <template>
     <div>
       <Calendar :events="events" @eventClick="handleEventClick" />
     </div>
   </template>

   <script>
   export default {
     data() {
       return {
         events: [
           {
             id: 1,
             name: 'Team Meeting',
             start: '2024-10-01T09:00:00',
             end: '2024-10-01T10:00:00',
             allday: false,
             type: 'primary',
           },
           {
             id: 2,
             name: 'Project Deadline',
             start: '2024-10-05',
             allday: true,
             type: 'danger',
           },
         ],
       };
     },
     methods: {
       handleEventClick(eventId) {
         console.log(`Clicked on event with ID: ${eventId}`);
       },
     },
   };
   </script>
   ```

## Props

| Prop        | Type     | Default | Description                                                      |
|-------------|----------|---------|------------------------------------------------------------------|
| `events`    | `Array`  | `[]`    | An array of event objects. Each event should have an `id`, `name`, `start`, `end`, `allday`, and `type`. |
| `eventClick` | `Function` | `() => {}` | A callback function triggered when an event is clicked. Receives the event's `id` as an argument. |

## Event Object Structure

Each event in the `events` array should follow this structure:

```js
{
  id: Number,                // Unique event ID
  name: String,              // Event name
  start: String,             // Event start time (ISO string format)
  end: String,               // Event end time (ISO string format)
  allday: Boolean,           // True for all-day events
  type: String               // Bootstrap color classes (e.g., 'primary', 'danger', 'success')
}
```

## License

This project is licensed under the **MIT License**. Feel free to use and modify the code as per the terms.

---

### Enjoy using this Vue Calendar component! 🎉
```

You can copy and paste this `README.md` file directly into your GitHub repository. It includes information on how to use the calendar component, installation steps, and details on the props and event structure.
