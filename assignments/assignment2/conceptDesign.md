# Concept Design

### Concept Specifications

**concept** ScheduleGenerator[User, Time, RepeatTime, Date, Percent]\
**purpose** manages events and tasks for users to automatically generate a schedule that meets their needs
**principle** Given a set of events and tasks, an optimal schedule for the user is created. When events and tasks are updated and removed, the schedule is regenerated

**state**

    a set of Schedules with
        an owner User
        a set of Events
        a set of Tasks

    a set of Events with
        a schedulePointer Schedule
        a startTime Time
        an endTime Time
        a repeatTime RepeatTime

    a set of Tasks with
        a schedulePointer Schedule
        a deadline Date
        an expectedCompletionTime Number
        a completionLevel Percent
        a priority Percent

**actions**

    addEvent(schedule: Schedule, startTime: Time, endTime: Time, repeatTime: RepeatTime): (event: Event)
        requires: schedule exists
        effects: creates and returns an event to add to the set of events in schedule with the given attributes, with schedulePointer pointing to schedule

    editEvent(schedule: Schedule, oldEvent: Event, startTime: Time, endTime: Time, repeatTime: RepeatTime)
        requires: oldEvent is in the set of Events of schedule
        effects: modifies oldEvent in the set of Events in schedule with the given attributes

    deleteEvent(schedule: Schedule, event: Event)
        requires: event is in the set of Events of schedule
        effects: deletes event in the set of Events in schedule

    addTask(schedule: Schedule, deadline: Date, expectedCompletionTime: Number, priority: Percent): (task: Task)
        requires: schedule exists
        effects: returns and adds task to the set of tasks in schedule with the given attributes and 0% for completionLevel, with schedulePointer pointing to schedule

    editTask(schedule: Schedule, oldTask: Task, deadline: Date, expectedCompletionTime: Number, completionLevel: Percent priority: Percent)
        requires: oldTask is in the set of Tasks of schedule
        effects: modifies oldTasks in the set of Events in schedule with the given attributes

    deleteTask(schedule: Schedule, task: Task)
        requires: task is in the set of Tasks of schedule
        effects: deletes task in the set of Tasks in schedule

    system generateSchedule(schedule: Schedule, events: set of Events, tasks: set of Tasks): (newSchedule: Schedule | e: Error)
        requires: any of the above actions in ScheduleGenerator were performed

        effects:
            Creates newSchedule for schedule.owner such that if possible, all given events start, end, and repeat as specified, and task scheduling is optimized by its attributes.

            Generally, tasks with a sooner deadline, higher priority level, higher expectedCompletionTime, and higher completionTime are scheduled first.

            If doing this is not possible, then return an error.

---

**concept** EventSharing[User, Event, Schedule, Time, RepeatTime]\
 **purpose** Allows users to share their event times with other users, encouraging greater collaboration and better task management
**principle** Users can choose which events to make shareable and with whom, if they accept; shared users can make requests to change event times, which the owner accepts

**state**

    a set of SharedEvents with
        an event Event
        an owner User
        a set of sharedWith Users
        a startTime Time
        an endTime Time
        a repeatTime repeatTime
        a set of Requests

    a set of Requests with
        a requester User
        a startTime Time
        an endTime Time
        a repeatTime repeatTime

**actions**

    makeEventShareable(owner: User, event: Event): (sharedEvent: SharedEvent)
        requires: event exists
        effects: creates and returns a new SharedEvent using the same startTime, endTime, and repeatTime attributes as event, and requests as the empty set

    shareEventWith(sharedEvent: SharedEvent, target: User): (sharedEvent: sharedEvent)
        requires: target is not a sharedWith User of sharedEvent
        effects: adds target to the sharedWith set of event, and returns the sharedEvent

    unshareEventWith(sharedEvent: SharedEvent, target: User)
        requires: target is a sharedWith User of sharedEvent
        effects: removes target from the sharedWith set of sharedEvent

    acceptNewEvent(user: User, schedule: Schedule, sharedEvent: SharedEvent)
        requires: user is in the set of sharedWith in sharedEvent, and schedule has owner user

    rejectNewEvent(user: User, sharedEvent: SharedEvent)
        requires: user is in the set of sharedWith in sharedEvent
        effects: removes user from the sharedWith set of sharedEvent

    requestNewTime(user: User, sharedEvent: SharedEvent, startTime: Time, endTime: Time, repeatTime: RepeatTime): (newRequest: Request)
        requires: user is a sharedWith User of sharedEvent
        effects: creates and returns newRequest with given startTime, endTime, and repeatTime, and makes user its requester, and adds it to the set of requests in sharedEvent

    confirmNewTime(owner: User, schedule: Schedule sharedEvent: SharedEvent, request: Request)
        requires: owner is the owner of sharedEvent, schedule has owner as owner, and request is a part of its set of Requests
        effects: deletes request from set of Requests in sharedEvent

    rejectNewTime(owner: User, sharedEvent: SharedEvent, request: Request)
        requires: owner is the owner of sharedEvent and request is a part of its set of Requests
        effects: deletes request from the set of Requests in sharedEvent

---

**concept** ProgressNotification[User, Task, Percent]\
**purpose** continuously notify students so that they update their task completion progress.
**principle** every day, the app asks the user to update their progress completion level for each task, which prompts the schedule to re-generate using the new completion level percentages.

**state**

    a set of ProgressReports with
        an owner User
        a task Task
        a completionLevel Percent

**actions**

    system notifyForProgress(schedule: Schedule, owner: User)
        requires: current time is 8PM
        effects: sends a notification to owner so that they update their completionLevel of tasks in schedule

    updateProgress(task: Task, completionLevel: Percent): (completionLevel: Percent)
        requires: task exists and notifyForProgress has run

---

**concept** UserAuthentication[User]\
**purpose** limit access to known users and find users by name\
**principle** After a user registers with a username and a password, they can authenticate with that same username and password and be treated each time as the same user. They can also be looked up by other users when sharing events

**state**

    a set of Users with:
        a username String
        a password String

**actions**

    register (username: String, password: String): (user: User)
        requires: no User with username exists
        effects: create and return a new User with the given username and password

    authenticate (username: String, password: String): (user: User)
        requires: User with the same username and password exists
        effects: grants access to the User associated with that username and password

    verify (username: String): (verified: Boolean)
        requires: User with the given username exists
        effects: returns true if username exists, false otherwise

### Essential Synchronizations

    sync VerifySharedUser

        when EventSharing.shareEventWith (sharedEvent, target: User):
        where !UserAuthentication.verify(target: String)
        then throw (e: Error)

    sync AcceptingSharedEvent

        when EventSharing.acceptNewEvent (user, schedule, sharedEvent)
        then
            ScheduleGenerator.addEvent (schedule, sharedEvent.event, request.startTime, request.endTime, request.repeatTime)

            (ScheduleGenerator.generateSchedule for user is automatically run)

    sync OwnerUpdatesSharedEvent

        when ScheduleGenerator.editEvent (ownerSchedule: Schedule, event, startTime, endTime, repeatTime)
        where event is a part of a sharedEvent that has a set of sharedWith users
        then
            for each user in sharedWith of that sharedEvent:
                ScheduleGenerator.editEvent (userSchedule: Schedule, event, startTime, endTime, repeatTime)

                (ScheduleGenerator.generateSchedule for user is automatically run)


    sync OwnerConfirmsRequest

        when EventSharing.confirmNewTime (owner, ownerSchedule: Schedule, sharedEvent, request)
        then
            ScheduleGenerator.editEvent (ownerSchedule, sharedEvent.event, request.startTime, request.endTime, request.repeatTime)

            (ScheduleGenerator.generateSchedule for user is automatically run)

            (OwnerUpdatesSharedEvent sync is also run)


    sync UpdateAfterProgressReport

        when ProgressNotification.updateProgress (updatedTask: Task, completionLevel)
        then
            ScheduleGenerator.editTask (task.schedulePointer, updatedTask, completionLevel)

            (ScheduleGenerator.generateSchedule for user is automatically run)

### Role concepts play

1. ScheduleGenerator
   - ScheduleGenerator manages the creation and modification of schedules of events and tasks for users. As for its generic parameters, the User represents the authenticated owner of schedules, Time represents the time of day, RepeatTime specifies how often events repeat (e.g., Once every Monday, Weekends biweekly, or Never), Date represents a date on the calendar, and Percent represents a decimal between 0 and 1.
2. EventSharing
   - EventSharing contains the state and actions responsible for sharing events with other users, and communicating with them about their times. The User, Event, Schedule, Time, and RepeatTime generic parameters have the same meaning as they do in the ScheduleGenerator concept. The syncs with EventSharing ensure that changes made to SharedEvents are reflected in the events of the owner and the shared users, which in turn re-generates their schedules.
3. ProgressNotification
   - ProgressNotification ensures that Users are reminded to update the completionLevels of their tasks, so that the sync with ScheduleGenerator can dynamically update the User's schedule every day, depending on the user's progress. The User, Task, and Percent generic parameters have the same meaning as they do in ScheduleGenerator.
4. UserAuthentication
   - UserAuthentication ensures that each registered and authenticated User has their own set of Schedules that they have access to. The verify action is used in a sync with EventSharing to verify the identity of the target user when sharing an event. The User generic type here is used in the other concepts as the owner or recipient of events and tasks.
