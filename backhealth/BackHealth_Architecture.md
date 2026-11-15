# BackHealth — Architecture Diagrams

---

## 1. Overall System Architecture (All Phases)

```mermaid
flowchart TD
    subgraph ANDROID["📱 Android App — Kotlin (Phase 1)"]
        UI["UI Layer\nActivities + XML Layouts"]
        VM["ViewModel Layer\nStateFlow / LiveData"]
        REPO["Repository Layer"]
        ROOM["Room DB\nExercises · Logs · Prefs"]
        ALARM["Alarm Engine\nAlarmManager + Foreground Service"]
        ASSETS["Bundled Assets\nexercises_default.json + GIFs"]
    end

    subgraph BACKEND["☁️ Backend — Java Spring Boot (Phase 1.5)"]
        API["REST API\n/exercises · /exercises/version"]
        ADMINDB["Database\nExercise Library"]
        CDN["GIF / Media Storage"]
    end

    subgraph ADMINPANEL["🖥️ Admin Web Panel — React (Phase 1.5)"]
        ADMINUI["Admin Dashboard\nAdd · Edit · Disable · Reorder exercises"]
    end

    UI <--> VM
    VM <--> REPO
    REPO <--> ROOM
    REPO -. "Phase 1.5: sync on launch" .-> API
    ALARM --> UI
    ASSETS -. "MVP: seed data" .-> ROOM

    ADMINUI <--> API
    API <--> ADMINDB
    API <--> CDN
    ROOM -. "cache GIFs locally" .-> CDN
```

---

## 2. Android App — Internal Layer Architecture (MVVM)

```mermaid
flowchart TD
    subgraph UI["UI Layer — Activities"]
        HOME["HomeActivity"]
        EXALARM["ExerciseAlarmActivity\n(Full-screen)"]
        WALKALARM["WalkAlarmActivity\n(Full-screen)"]
        ONBOARD["OnboardingActivity"]
        LIBRARY["ExerciseLibraryActivity"]
        PROGRESS["ProgressActivity"]
        SETTINGS["SettingsActivity"]
    end

    subgraph VM["ViewModel Layer"]
        HVM["HomeViewModel\nNext alarm · Today stats · Streak"]
        LVM["ExerciseLibraryViewModel\nEnabled exercises · Exclusions"]
    end

    subgraph REPO["Repository Layer"]
        EXREPO["ExerciseRepository\nOffline-first · Sync logic"]
        PROGREPO["ProgressRepository\nLogs · Streaks · Stats"]
    end

    subgraph DB["Room Database"]
        EXTBL["exercise"]
        EXCLTBL["exercise_exclusion"]
        LOGTBL["exercise_log"]
        WALKTBL["walk_log"]
        PREFTBL["user_prefs"]
        CACHEMETA["exercise_cache_meta"]
    end

    subgraph ENGINE["Alarm Engine"]
        SVC["AlarmCycleService\nForeground Service"]
        RECV["AlarmReceiver\nBroadcastReceiver"]
        BOOT["BootReceiver\nBoot Completed"]
        SCHED["AlarmScheduler\nAlarmManager wrapper"]
        PICKER["ExercisePicker\nRandom selection"]
        SLEEP["SleepWindowChecker\nWake · Bed time guard"]
    end

    HOME --> HVM
    LIBRARY --> LVM
    HVM --> PROGREPO
    LVM --> EXREPO
    EXREPO --> EXTBL
    EXREPO --> EXCLTBL
    EXREPO --> CACHEMETA
    PROGREPO --> LOGTBL
    PROGREPO --> WALKTBL
    PROGREPO --> PREFTBL

    SVC --> SCHED
    SVC --> PICKER
    SVC --> SLEEP
    RECV --> SVC
    BOOT --> SVC
    PICKER --> EXTBL
    RECV --> EXALARM
    RECV --> WALKALARM
    EXALARM --> LOGTBL
    WALKALARM --> WALKTBL
```

---

## 3. Alarm Cycle State Machine

```mermaid
stateDiagram-v2
    [*] --> IDLE : App start / Rest ends / Boot

    IDLE --> EXERCISE_PENDING : AlarmManager fires\n(T + 20 min)

    EXERCISE_PENDING --> IDLE : User taps Done\n→ schedule Walk alarm (T + 40 min)

    EXERCISE_PENDING --> EXERCISE_PENDING : User taps Snooze\n→ re-fire in 2 min

    IDLE --> WALK_PENDING : AlarmManager fires\n(T + 40 min)

    WALK_PENDING --> IDLE : User taps "I'm Walking"\n→ 5-min walk timer\n→ schedule Exercise alarm (T + 20 min)

    WALK_PENDING --> WALK_PENDING : User taps Snooze\n→ re-fire in 2 min

    IDLE --> REST_MODE : User taps "Take a Rest"
    EXERCISE_PENDING --> REST_MODE : User taps "Take a Rest"
    WALK_PENDING --> REST_MODE : User taps "Take a Rest"

    REST_MODE --> IDLE : Rest duration ends\n(auto-resume)

    IDLE --> SLEEP_WINDOW : Current time outside\nwake–bed window
    SLEEP_WINDOW --> IDLE : Wake-up time reached
```

---

## 4. Alarm Engine — Trigger & Delivery Flow

```mermaid
sequenceDiagram
    participant ACS as AlarmCycleService
    participant AM as AlarmManager
    participant AR as AlarmReceiver
    participant EP as ExercisePicker
    participant DB as Room DB
    participant ACT as ExerciseAlarmActivity
    participant USER as User

    ACS->>AM: setExactAndAllowWhileIdle(T+20min)
    Note over AM: Phone may be asleep / Doze mode
    AM-->>AR: onReceive() fires at exact time
    AR->>EP: pickRandom(SET_1), pickRandom(SET_2)
    EP->>DB: getEnabledBySet() [coroutine / IO thread]
    DB-->>EP: Exercise list
    EP-->>AR: Exercise 1 + Exercise 2
    AR->>ACT: startActivity (full-screen intent)
    ACT-->>USER: Full-screen alarm shown (even on lock screen)

    alt User taps Done
        USER->>ACT: Done
        ACT->>DB: insert ExerciseLog (completed=1)
        ACT->>ACS: onTaskComplete(EXERCISE)
        ACS->>AM: setExactAndAllowWhileIdle(T+40min)
    else User taps Snooze
        USER->>ACT: Snooze
        ACT->>ACS: onSnooze()
        ACS->>AM: setExactAndAllowWhileIdle(T+2min)
    end
```

---

## 5. Exercise Sync Flow — Offline-First (Phase 1.5)

```mermaid
flowchart TD
    START([App Launch]) --> CHECK["ExerciseSyncManager\nCheck local cache version"]
    CHECK --> COMPARE{Remote version\n> Local version?}
    COMPARE -- No --> LOADLOCAL["Load exercises\nfrom Room DB"]
    COMPARE -- Yes --> FETCH["GET /exercises\nfrom Spring Boot API"]
    FETCH --> STORE["Replace all exercises\nin Room DB\nUpdate cache version"]
    STORE --> LOADLOCAL
    LOADLOCAL --> PICKER["ExercisePicker\nFilters out admin-disabled\n+ user-excluded exercises"]
    PICKER --> ALARM["Ready for\nnext alarm cycle"]

    NOINTERNET([No Internet]) --> LOADLOCAL
```

---

## 6. Rest Mode & Sleep Window Flow

```mermaid
flowchart TD
    USER([User]) -->|Taps "Take a Rest"| BOTTOM["Bottom Sheet\n20 min · 30 min · 40 min\n1 hr · 1.5 hr · Custom"]
    BOTTOM --> CANCEL["Cancel all\npending AlarmManager alarms"]
    CANCEL --> NOTIFY["Show persistent notification\n'Resting — resumes in X min'"]
    NOTIFY --> TIMER["Schedule rest-end alarm\nvia AlarmManager"]
    TIMER --> RESTEND([Rest duration ends])
    RESTEND --> RESUME["Restart AlarmCycleService\nState → IDLE\nSchedule T+20min exercise alarm"]

    SCHED([Every alarm about to fire]) --> SLEEPCHECK{"Current time\ninside wake–bed\nwindow?"}
    SLEEPCHECK -- Yes --> FIRE["Fire alarm normally"]
    SLEEPCHECK -- No --> SKIP["Skip alarm silently\nReschedule for next\nvalid window"]
```
