# Lychee-Schedules
Lychee Schedules 
Auto-Scheduling Calendar
https://lychee-schedules.lovable.app

Lychee Schedules is a website that automatically schedules any tasks or events you have to conveniently plan out your day. This website is aimed at busy students or workers that have no time to worry about when to do what. It’s very useful when you know what you have to do but don’t know when to do it. It can add your tasks and events from Google Calendar as well so you don’t have to move them over yourself. 

Functionalities:
1. Tasks are pulled from Google Tasks (synchronized with Google Calendar)
2. Events are pulled from Google Calendar (all day events included!)
3. Tasks/events added to the To-Do List will be automatically scheduled onto the calendar based on your settings
4. Use Re-plan to shuffle around the order of the tasks / reschedule the tasks after making edits to settings
5. Settings have three modes of autoscheduling: Efficiency mode - tasks are packed into the earliest available slots and spaced by the minimum buffer time; Equal mode - caps how many tasks there are per day and spreads them evenly across working hours (extra tasks roll over to the next available day); Flexible mode - you can set per-day task and hour caps as well as decide whether tasks are spread evenly or by the minimum buffer time
7. Settings also have working hour ranges and off hours that all scheduling modes will adhere to.
8. The home page shows a preview of your calendar, tasks, and events for the day.
9. The To-Do List has a filtering system where you can sort tasks by difficulty, type, tags, time range, and recurrence. Of course, you can also make tasks with these features. 


An example scenario of how data can be transmitted: The user creates a task in Google Tasks (which they have already connected to Lychee Schedules via the Google Calendar API which is part of external services). This sync is one of the edge functions that occur. The change is saved to the database and then the task is displayed to the user via the Auto-Scheduler UI on the Calendar tab. 

Code Snippets

FLEXIBLE MODE

    const flexMaxTasksPerDay: Record<string, number> =
              mode === "flexible" && prefs.flex_max_tasks_per_day && typeof prefs.flex_max_tasks_per_day === "object"
                ? Object.fromEntries(
                    Object.entries(prefs.flex_max_tasks_per_day as Record<string, unknown>)
                      .filter(([, v]) => typeof v === "number" && (v as number) > 0)
                      .map(([k, v]) => [k, v as number])
                  )
                : {};
                
Records the maximum number of tasks per day in Flexible Mode and verifies that the data is not invalid and/or the wrong type

        const flexMaxHoursPerDay: Record<string, number> =
          mode === "flexible" && prefs.flex_max_hours_per_day && typeof prefs.flex_max_hours_per_day === "object"
            ? Object.fromEntries(
                Object.entries(prefs.flex_max_hours_per_day as Record<string, unknown>)
                  .filter(([, v]) => typeof v === "number" && (v as number) > 0)
                  .map(([k, v]) => [k, v as number])
              )
            : {};

Records the maximum number of hours of tasks per day in Flexible Mode and verifies that the data is not invalid and/or the wrong type

        const flexDayOverrides: Record<
          string,
          {
            working_hours?: { start?: string; end?: string } | null;
            off_hours?: Array<{ start?: string; end?: string }> | null;
          }
        > =
          mode === "flexible" && prefs.flex_day_overrides && typeof prefs.flex_day_overrides === "object"
            ? (prefs.flex_day_overrides as Record<string, {
                working_hours?: { start?: string; end?: string } | null;
                off_hours?: Array<{ start?: string; end?: string }> | null;
              }>)
            : {};

Makes sure the working hours and off hours set by the user take priority and that the Flexible Mode tasks do not override them

               const spaceEvenly = mode === "flexible" && prefs.flex_space_evenly === true;

If the user chooses to space out the tasks evenly in Flexible Mode, it does so

AUTOSCHEDULER CODE

        const { data: scheduledEvents } = await supabase
          .from("events")
          .select("scheduled_start, scheduled_end")
          .eq("user_id", userId)
          .or("all_day.is.null,all_day.eq.false");

Fetches the already scheduled events and their time blocks (excluding the all-day events that just have a banner) that shouldn’t be scheduled over

        const { data: externalEvents } = await supabase
          .from("external_events")
          .select("scheduled_start, scheduled_end")
          .eq("user_id", userId)
          .or("all_day.is.null,all_day.eq.false");

Fetches the events from Google Calendar and their time blocks (excluding the all-day events that just have a banner) that shouldn’t be scheduled over

        const occupied: { start: number; end: number }[] = [];
        for (const item of [...(scheduledTasks || []), ...(scheduledEvents || []), ...(externalEvents || [])]) {
          if (item.scheduled_start && item.scheduled_end) {
            occupied.push({
              start: new Date(item.scheduled_start).getTime(),
              end: new Date(item.scheduled_end).getTime(),
            });
          }
        }

Creates an array representing busy time-periods and loops through the three occupied sources (scheduled tasks, scheduled events, Google Calendar events), adding the time-intervals of these sources to the array

