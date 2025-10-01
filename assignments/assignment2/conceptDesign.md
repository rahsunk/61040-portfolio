# Concept Design

### Concept Specifications

**concept** ScheduleGenerator[User, Time, RepeatTime, Date, Percent]\
**purpose** manages events and tasks for users to generate a schedule that meets their needs
**principle** Given a set of events and tasks, an optimal schedule for the user is created. When events and tasks are updated and removed, the schedule is regenerated

**state**

    a set of Schedules with
        an owner User
        a set of Events
        a set of Tasks

    a set of Events with
        a startTime Time
        an endTime Time
        a repeatTime RepeatTime

    a set of Tasks with
        a deadline Date
        an expectedCompletionTime Number
        a completionLevel Percent
        a priority Percent

**actions**

    addEvent(schedule: Schedule, startTime: Time, endTime: Time, repeatTime: RepeatTime): (event: Event)
        requires: schedule exists
        effects: creates and returns an event to add to the set of vents in schedule with the given attributes

    editEvent(schedule: Schedule, oldEvent: Event, startTime: Time, endTime: Time, repeatTime: RepeatTime)
        requires: oldEvent is in the set of Events of schedule
        effects: modifies oldEvent in the set of Events in schedule with the given attributes

    deleteEvent(schedule: Schedule, event: Event)
        requires: event is in the set of Events of schedule
        effects: deletes event in the set of Events in schedule

    addTask(schedule: Schedule, deadline: Date, expectedCompletionTime: Number, priority: Percent)
        requires: schedule exists
        effects: adds task to set of tasks in schedule with the given attributes and 0% for completionLevel

    editTask(schedule: Schedule, oldTask: Task, deadline: Date, expectedCompletionTime: Number, completionLevel: Percent priority: Percent)
        requires: oldTask is in the set of Tasks of schedule
        effects: modifies oldTasks in the set of Events in schedule with the given attributes

    deleteTask(schedule: Schedule, task: Task)
        requires: task is in the set of Tasks of schedule
        effects: deletes task in the set of Tasks in schedule

    generateSchedule(owner: User, events: set of Events, tasks: set of Tasks): (newSchedule: Schedule | e: Error)
        requires: owner, events, and tasks exist
        effects:
            creates newSchedule for owner such that if possible, all events start, end, and repeat as specified, and task scheduling is optimized by its attributes. Generally, tasks with a sooner deadline, higher priority level, higher expectedCompletionTime, and less completionTime are scheduled first.

            If doing this is not possible, then return an error.

---

**concept** EventSharing[User, Time, RepeatTime]\
 **purpose** Allows users to share their event times with other users, encouraging greater collaboration and better task management
**principle** Users can choose which events to make shareable and with who, if they accept; shared users can make requests to change event times which the owner accepts

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
        effects: creates and returns a new SharedEvent using the same startTime, endTime, and repeatTime attributes as event, and initializing owner as the given owner, and requests as the empty set

    makeEventUnshareable(sharedEvent: SharedEvent)
        requires: sharedEvent exists and its set of sharedWith users is empty
        effects: deletes sharedEvent, i.e. this event is no longer able to be shared.

    shareEventWith(sharedEvent: SharedEvent, target: User): (sharedEvent: sharedEvent)
        requires: target exists and is not a sharedWith User of sharedEvent
        effects: adds target to sharedWith set of event, and returns the sharedEvent

    unshareEventWith(sharedEvent: SharedEvent, target: User)
        requires: target is a sharedWith User of sharedEvent
        effects: removes target from sharedWith set of sharedEvent

    acceptNewEvent(user: User, sharedEvent: SharedEvent)
        requires: user is in the set of sharedWith in sharedEvent

    rejectNewEvent(user: User, sharedEvent: SharedEvent)
        requires: user is in the set of sharedWith in sharedEvent
        effects: removes user from sharedWith set of sharedEvent

    getSharedOwnerEvents(owner: User): (sharedEvents: set of SharedEvents)
        requires: owner exists
        effects: returns the set of SharedEvents that owner is an owner of

    getEventsSharedWithMe(user: User): (sharedEvents: set of SharedEvents)
        requires: user exists
        effects: returns the set of SharedEvents that user is a part of their set of sharedWith Users

    requestNewTime(user: User, sharedEvent: SharedEvent, startTime: Time, endTime: Time, repeatTime: RepeatTime): (newRequest: Request)
        requires: user is a sharedWith User of sharedEvent; newStart and newEnd exist
        effects: creates and returns newRequest with given startTime, endTime, and repeatTime, makes user its requester, and adds it to set of requests in sharedEvent

    confirmNewTime(owner: User, sharedEvent: SharedEvent, request: Request)
        requires: owner is the owner of sharedEvent and request is a part of its set of Requests
        effects: deletes request from set of Requests in sharedEvent

    rejectNewTime(owner: User, sharedEvent: SharedEvent, request: Request)
        requires: owner is the owner of sharedEvent and request is a part of its set of Requests
        effects: deletes request from set of Requests in sharedEvent

---

**concept** ProgressReporting[User, Percent]\
**purpose** Track and update task completion progress reported by users.
**principle** Every day, the app asks the user to update their progress completion level for each task, which prompts the schedule to re-generate using the new completion level percentages.

**state**

    a set of ProgressReports with
        an owner User
        a task Task
        a completionLevel Percent

**actions**

    getProgress(task: Task): (completionLevel: Percent)
        requires: task exists
        effects: returns the completionLevel of the given task

    system updateProgress(task: Task, completionLevel: Percent): (completionLevel: Percent)
        requires: task exists and the current time is 8PM
        effects: sends a notification to Users to update the completionLevel of task, and returns the new user-inputted completionLevel

---

**concept** UserAuthentication[User]\
**purpose** limit access to known users\
**principle** after a user registers with a username and a password, they can authenticate with that same username and password and be treated each time as the same user

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

### Essential Synchronizations

    sync UpdateAfterAcceptingSharedEvent

        when EventSharing.acceptNewEvent (user, sharedEvent)
        then
            ScheduleGenerator.addEvent (user's schedule, sharedEvent.event, request.startTime, request.endTime, request.repeatTime)
            ScheduleGenerator.generateSchedule(owner): (newSchedule: Schedule | e: Error)

    sync OwnerUpdatesSharedEvent

        when ScheduleGenerator.editEvent (owner's schedule, event, startTime, endTime, repeatTime)
        where event is a part of a sharedEvent with sharedWith users
        then
            for each user in sharedWith of that sharedEvent:
                ScheduleGenerator.editEvent (user's schedule, event, startTime, endTime, repeatTime)
                ScheduleGenerator.generateSchedule(user): (newSchedule: Schedule | e: Error)

    sync ConfirmNewRequestTime

        when EventSharing.confirmNewTime (owner, sharedEvent, request)
        then
            ScheduleGenerator.editEvent (owner's schedule, sharedEvent.event, request.startTime, request.endTime, request.repeatTime)
            ScheduleGenerator.generateSchedule(owner): (newSchedule: Schedule | e: Error)

    sync UpdateAfterProgressReport

        when System.ProgressReporting.updateProgress (updatedTask: Task, completionLevel)
        then
            ScheduleGenerator.editTask (owner's schedule, updatedTask, completionLevel)
            ScheduleGenerator.generateSchedule(owner of tasks, owner.events, owner.tasks): (newSchedule: Schedule | e: Error)

### Role concepts play

1. ScheduleGenerator
   - ScheduleGenerator manages the creation and modification of schedules of events and tasks for users. As for its generic parameters, the User represents the authenticated owner of schedules, Time reperesents the time of day, RepeatTime specifies how often events repeat (e.g. Once every Monday, Weekends biweekly), Date represents a date on the calendar, and Percent reprsents a decimal between 0 to 1.
2. EventSharing
   - EventSharing contains the state and actions responsible for sharing events with other users, and communicating with them about their times. The User, Time, and RepeatTime generic parameters have the same meaning as they do in ScheduleGenerator. The syncs with EventSharing ensure that changes made to SharedEvents are reflected in the events of the owner and the shared users, which in turn re-generates their schedules.
3. ProgressReporting
   - ProgressReporting ensures that Users are reminded to update the completionLevels of their tasks, so that the sync with ScheduleGenerator can dyanimcally update the User's schedule every day depending on the user's progress. The User and Percent generic parameters have the same meaning as they do in ScheduleGenerator.
4. UserAuthentication
   - UserAuthentication ensures that each registered and authenticated User has their own set of Schedules that they have access to. The User generic type here is used in the other concepts as the owner or recipient of events and tasks.
