# BackHealth — Android App Design Plan

> **Phase 1 Focus:** Lumbar (Lower Back) Health  
> **Target Users:** People with existing disc issues + preventive users  
> **Platform:** Android Native (Kotlin)  
> **Min Android Version:** API 35 / Android 15

---

## 1. App Overview

**BackHealth** is a smart companion app that keeps desk workers and people with lumbar disc issues moving throughout the day. It uses an intelligent alarm cycle to prompt micro-exercises and walking breaks, respecting the user's sleep/rest schedule.

---

## 2. User Personas & Targeted Routines

| Persona | Description | Daily Alarm & Break Routine |
|---|---|---|
| **Disc Patient** | Has a diagnosed lumbar disc issue (L4-L5, L5-S1 etc.) | **Alternating 20-min cycle**: 20m Micro-Exercises (Set 1 + Set 2) ↔ 20m Walking Breaks (5 min walk) |
| **Preventive User** | People who don't have any back issues | **40-min Walking Break**: Walk for 5 mins after every 40 mins. **No 20-min exercise alarm required.** |

> [!NOTE]
> - **Disc Patients**: Require frequent mechanical pumping and posture resetting every 20 minutes to prevent disc tear compression.
> - **Preventive Users**: Do not require micro-exercises; their primary requirement is breaking prolonged sitting by walking for 5 minutes every 40 minutes.

---

## 3. Core Alarm Cycles

### 3.1 Disc Patient Cycle (Lumbar Recovery)
```
[Start]
  │
  ├─ Alarm 1 @ 20 min → Small Break: Posture Reset Exercises (2–3 min window)
  │        └─ User taps Done → next 20m timer starts
  │
  ├─ Alarm 2 @ 20 min (40m mark) → Big Break: Walk for 3–5 min (5-min timer)
  │        └─ User taps "I'm Walking" → 5-min countdown → next 20m timer starts
  │
  ├─ Alarm 3 @ 20 min → Small Break: Posture Reset Exercises (2–3 min window)
  │        └─ User taps Done → next 20m timer starts
  │
  └─ Alarm 4 @ 20 min (80m mark) → Big Break: Walk for 3–5 min
           └─ Cycle repeats
```

### 3.2 Preventive User Cycle (No Back Issues)
```
[Start]
  │
  ├─ Alarm @ 40 min → Walk Break: Walk for 5 minutes (5-min timer)
  │        └─ User taps "I'm Walking" → 5-min countdown → next 40m timer starts
  │
  └─ [Repeats every 40 minutes throughout the active day — NO 20-min exercise alarm]
```

Each alarm timer **starts counting only after the user completes the previous task** (or after the allotted task time expires).

---

## 4. Exercise Library (Phase 1 — Lumbar)

For every **20-minute alarm**, the app randomly picks **one exercise from each set** (Set 1: Lower Body & Core Pump + Set 2: Upper Body & Posture Reset). The user loops the exercise continuously within the **2–3 minute window**.

> [!IMPORTANT]
> The exercise library is **admin-configurable**. Exercises, their descriptions, durations, coaching cues, and set assignments can be managed from an admin panel / backend without a new app release. The app always fetches the latest list on launch and caches it locally.

---

### 4.1 Set 1 — Lower Body & Core Pump

| # | Exercise | Reps / Duration | Key Coaching Cue |
|---|---|---|---|
| 1 | **Slow Standing March** | 30 seconds | Lift knees just high enough to engage hips — do NOT let the lower back round |
| 2 | **Calf Raises** | 20 reps | Hold desk for balance; push straight up onto toes |
| 3 | **Glute Squeezes** | 10 reps × 3-sec hold | Stand tall; consciously squeeze glutes to relieve lumbar stabiliser muscles |
| 4 | **Standing Wall Calf Stretch** | 15 sec per side | Back leg straight, heel flat on floor; keep spine in a rigid, straight line |
| 5 | **Standing Quad Stretch** | 15 sec per side | Knees together, torso upright — do NOT lean forward |
| 6 | **Standing Hip Flexor Stretch** | 30 sec per side | ⚠️ Brace stomach muscles tightly — if stomach sags, lower back arches and can pinch the disc tear |
| 7 | **Standing Ankle Circles** | 10 reps per side | Stand fully upright, hold desk; keep spine fully decompressed |

**Purpose of Set 1:** Acts as a gentle, low-impact mechanical pump to clear synovial fluid from the L4-L5 joint and reduce lumbar muscle fatigue from sustained sitting.

---

### 4.2 Set 2 — Upper Body & Posture Reset

| # | Exercise | Reps / Duration | Key Coaching Cue |
|---|---|---|---|
| 1 | **Standing Chin Tucks** | 10 reps | Pull chin straight back horizontally (double-chin motion) — do NOT tilt down |
| 2 | **Exaggerated Backward Shoulder Rolls** | 5 reps | Shrug up → roll far back → pull firmly down; un-pinches the brachial plexus nerves |
| 3 | **Standing Shoulder Blade Squeezes** | 10 reps × 3-sec hold | Pull shoulder blades together as if holding a pencil between them |
| 4 | **Standing Upper Trap Stretch** | 15 sec per side | Tilt ear toward same-side shoulder gently; feel stretch on opposite side of neck |
| 5 | **Doorway Chest Opener** | 15 sec per side | Forearm on doorframe, elbow at 90°; ⚠️ keep abs tight so lower back does NOT arch |
| 6 | **Vertical Overhead Reach** | 10 seconds | Fingers interlocked, reach straight up; push strictly upward — do NOT lean backward |
| 7 | **Wall Angels** | 10 reps | Back against wall, slide arms up and down; natural stance — do NOT force lower back flat or aggressively arch it |

**Purpose of Set 2:** Reverses the forward-head, rounded-shoulder posture that builds up during desk work. Realigns the cervical spine and relieves tension in the upper back and brachial plexus.

---

> [!TIP]
> Each exercise displays a **high-definition looping demonstration video (H.264 MP4 / WebM) + step-by-step coaching cues**:
> - **Clear Movement Demonstration**: Full-motion looping video showing proper form, tempo, and joint angles from optimal camera angles.
> - **Form & Safety Cues**: Highlighted coaching instructions (e.g., bracing the core, avoiding lumbar rounding, keeping neck aligned).
> - **Fast Mobile Playback**: Seamless, muted hardware-accelerated playback with zero latency via `VideoView` / ExoPlayer.
> Users can mark individual exercises as "too painful" or "not suitable" to exclude them from rotation — this feeds back to the admin analytics.
> The exercise loops automatically within the 2–3 min window (e.g., if one round of Glute Squeezes takes ~40 sec, the user does it ~3–4 times before Done appears).

---

## 5. Walk Breaks

### 5.1 Disc Patients (Big Break — 5 Minutes)
- **Schedule**: Alternating 20-minute mark (every even interval, e.g. 40m, 80m).
- Full-screen alarm card: **"🚶 Big Break"**
- User taps **"I'm Walking"** → 5-minute countdown begins on screen.
- After 5 minutes → notification rings → next 20-min Small Break (exercises) cycle begins.

### 5.2 Preventive Users / No Back Issues (Walk Break — 5 Minutes)
- **Schedule**: Every 40 minutes (no micro-exercise break).
- Full-screen alarm card: **"🚶 Walk Break"**
- User taps **"I'm Walking"** → 5-minute (300s) countdown begins on screen.
- After walking → notification rings → next 40-minute walk timer begins.
- If user doesn't tap, snooze prompt appears after 2 minutes.
- *(Phase 2: auto-detect via step counter)*

---

## 6. Alarm Behavior & Controls

### 6.1 Alarm Style
- **Full-screen alarm activity** — appears even when phone is locked/screen off
- Requires `USE_FULL_SCREEN_INTENT` permission (Android 14+: must be explicitly granted)
- Accompanied by sound + vibration

### 6.2 Snooze

- Snooze button on every alarm screen
- Snooze interval: **every 2 minutes**, re-fires until task is marked Done
- Snooze does **not** reset the upcoming alarm cycle timer

### 6.3 Rest / Pause Mode

- **"Take a Rest"** button always accessible from home screen
- Tapping opens bottom sheet with options:

| Option | Duration |
|---|---|
| Short nap | 20 minutes |
| Power nap | 30 minutes |
| Rest | 40 minutes |
| Long rest | 1 hour |
| Extended rest | 1.5 hours |
| Custom | User-defined |

- During rest: **all alarms silenced**
- After rest: **cycle restarts automatically** from the 20-min countdown
- Persistent notification shows remaining rest time + **"Wake me up early"** action

### 6.4 Sleep Window

- User sets **Wake-up time** and **Bedtime** in Settings
- Zero alarms fire outside this window
- If an alarm would fire within 10 minutes of bedtime, it is silently skipped

---

## 7. Tech Stack

### 7.1 Mobile App — Kotlin Native Android

| Layer | Choice |
|---|---|
| **Language** | Kotlin |
| **UI** | XML Layouts + View Binding |
| **Architecture** | MVVM (ViewModel + LiveData / StateFlow) |
| **Async** | Kotlin Coroutines |
| **Alarms** | AlarmManager (Exact Alarms) |
| **Full-screen alarm** | Full-screen Intent Activity |
| **Background** | Foreground Service |
| **Notifications** | NotificationCompat |
| **Boot resilience** | BroadcastReceiver (BOOT_COMPLETED) |
| **Local DB** | Room |
| **Media & Video** | Android `VideoView` / ExoPlayer (Looping MP4/WebM Form Demos) + Coil/Glide |
| **DI** | Manual DI (MVP) → Hilt (Phase 2) |
| **HTTP client** | Retrofit + Gson (Phase 1.5 onwards) |
| **Build script** | Gradle Kotlin DSL (`build.gradle.kts`) |

### 7.2 Backend API — Java Spring Boot *(Phase 1.5)*

| Layer | Choice |
|---|---|
| **Language** | Java |
| **Framework** | Spring Boot |
| **API style** | REST / JSON |
| **Responsibilities** | Exercise CRUD, Video loop CDN delivery, version control, analytics |

### 7.3 Admin Web Panel — React *(Phase 1.5)*

| Layer | Choice |
|---|---|
| **Language** | JavaScript / TypeScript |
| **Framework** | React |
| **Responsibilities** | Add / edit / disable exercises, upload demonstration video loops, view analytics |

### 7.4 Implementation Phases

| Phase | Scope | Stack Involved |
|---|---|---|
| **Phase 1** | Kotlin Android app — fully self-contained | Kotlin + Room + AlarmManager |
| **Phase 1.5** | Admin-configurable exercise system | + Spring Boot API + React Admin Panel |
| **Phase 2** | Cervical exercises, user-type intensity levels, Hilt DI | Kotlin Android + Spring Boot |
| **Phase 3** | Auto walk detection (step counter), personalised plans by disc level | Kotlin Android |
| **Phase 4** | Wearable integration, cloud user sync, doctor-shared routines | All layers |

---

## 8. Permissions Required

| Permission | Purpose | When Prompted |
|---|---|---|
| `POST_NOTIFICATIONS` | Show alarm notifications (Android 13+) | Onboarding |
| `USE_EXACT_ALARM` | Precise alarm timing | Onboarding |
| `SCHEDULE_EXACT_ALARM` | Fallback for exact alarms | Onboarding |
| `USE_FULL_SCREEN_INTENT` | Full-screen alarm activity | Onboarding |
| `FOREGROUND_SERVICE` | Keep alarm cycle alive | Auto (declared in manifest) |
| `FOREGROUND_SERVICE_SPECIAL_USE` | Required for Android 14+ foreground service | Manifest |
| `RECEIVE_BOOT_COMPLETED` | Restart alarms after reboot | Manifest |
| `VIBRATE` | Vibration on alarm | Manifest |
| `REQUEST_IGNORE_BATTERY_OPTIMIZATIONS` | Prevent OS from killing service | Onboarding (explained clearly) |

> [!IMPORTANT]
> On Android 15 (API 35), `USE_FULL_SCREEN_INTENT` is restricted to apps with a specific use case declaration. You must declare `<uses-permission android:name="android.permission.USE_FULL_SCREEN_INTENT" />` **and** handle the case where the user denies it gracefully (fall back to heads-up notification).

---

## 9. Screen & Navigation Design

### 9.1 App Navigation Map

```
App Launch
    │
    ├── Onboarding (first-time only)
    │       ├── User Type Selection (Disc Patient / Preventive)
    │       ├── Wake-up & Bedtime Setup
    │       ├── Permission Requests (with rationale)
    │       └── Exercise Preview Carousel
    │
    └── Main App
            ├── Home Screen (dashboard)
            ├── Full-Screen Exercise Alarm Activity
            ├── Full-Screen Walk Alarm Activity
            ├── Exercise Library (browse & disable exercises)
            ├── Progress / History Screen
            └── Settings Screen
```

### 9.2 Home Screen

```
┌──────────────────────────────────────────┐
│  🌿 BackHealth             [⚙ Settings]  │
│                                          │
│  ┌────────────────────────────────────┐  │
│  │  Next alarm in:   00:18:42         │  │
│  │  Type: 🧘 Posture Reset            │  │
│  └────────────────────────────────────┘  │
│                                          │
│  Today's Progress                        │
│  ✅ Exercises: 4   🚶 Walks: 2          │
│                                          │
│  ┌────────────────────────────────────┐  │
│  │  Streak: 🔥 3 days                 │  │
│  └────────────────────────────────────┘  │
│                                          │
│  [ 😴 Take a Rest ]  [ ⏸ Pause Day ]    │
│                                          │
└──────────────────────────────────────────┘
```

### 9.3 Full-Screen Exercise Alarm

```
┌──────────────────────────────────────────┐
│  ⏰ Small Break: Micro-Exercises         │
│  [ Tab 1: Lower Body ] [ Tab 2: Posture ]│
│                                          │
│  ┌────────────────────────────────────┐  │
│  │ 🎬 EXERCISE FORM DEMO (Loop)       │  │
│  │                                    │  │
│  │     [ Looping Video Demo ]         │  │
│  │    [ Clear Joint Movement ]        │  │
│  │                                    │  │
│  │ ▶ Play/Pause     🔁 Seamless Loop  │  │
│  └────────────────────────────────────┘  │
│                                          │
│  🦵 Slow Standing March (30s)            │
│  ──────────────────────────────          │
│  • Lift knees just high enough for hips  │
│  • ⚠️ Do not let lower back round!       │
│                                          │
│  [ ✅  Done ]     [ 😴 Snooze 2 min ]   │
└──────────────────────────────────────────┘
```

### 9.4 Full-Screen Walk Alarm

```
┌──────────────────────────────────────────┐
│  🚶 Walk Break Time!                     │
│                                          │
│  Take a 3–5 minute walk.                 │
│  Let your spine reset!                   │
│                                          │
│  [Walking illustration / GIF]            │
│                                          │
│  [ 🚶  I'm Walking! ]                   │
│                                          │
│  ──────── or ────────                    │
│                                          │
│  [ 😴 Snooze 2 min ]                    │
└──────────────────────────────────────────┘
```

---

## 10. Onboarding Flow

```
Screen 1 — Welcome
  "BackHealth helps you protect your spine
   with smart micro-breaks throughout your day."
  [ Get Started ]

Screen 2 — Who are you?
  ○ I have a lumbar disc issue
  ○ I want to prevent back problems
  [ Continue ]

Screen 3 — Set your day
  Wake-up time: [ 06:30 AM ] ▾
  Bedtime:      [ 10:30 PM ] ▾
  [ Continue ]

Screen 4 — Permissions
  "BackHealth needs a few permissions to remind
   you at the right time, even when the screen is off."
  → Notification permission
  → Exact alarm permission
  → Full-screen intent permission
  → Battery optimization exclusion
  [ Grant Permissions ]

Screen 5 — Exercise Preview
  Swipe through exercise cards (GIF + text)
  [ Looks Good! Start ]
```

---

## 11. Notification Design

| Trigger | Title | Body | Actions |
|---|---|---|---|
| 20-min alarm | "🧘 Posture Reset!" | "2-min exercise break — your spine will thank you" | Done · Snooze |
| 40-min alarm | "🚶 Walk Break!" | "Get up and walk for 3–5 minutes" | I'm Walking · Snooze |
| Snooze reminder | "⏰ Still waiting…" | "Your posture break is overdue!" | Done · Snooze |
| Rest ending | "😊 Rest Complete!" | "Ready to resume your spine-care routine?" | Resume · 10 more mins |
| Walk timer done | "✅ Great walk!" | "Back to work — next check-in in 20 mins" | — |

---

## 12. Settings Screen

| Setting | Type | Default |
|---|---|---|
| Wake-up time | Time picker | 07:00 AM |
| Bedtime | Time picker | 10:00 PM |
| 20-min exercise reminder | Toggle | ON |
| 40-min walk reminder | Toggle | ON |
| Sound on alarm | Toggle | ON |
| Vibration on alarm | Toggle | ON |
| User type | Selector | Preventive |
| Reset exercise exclusions | Button | — |
| App version / About | Info | — |

---

## 13. Alarm Engine — Kotlin Coroutines Service Logic

```
AlarmCycleService.kt  [Foreground Service]
    │
    ├─ State Machine: IDLE → EXERCISE_PENDING → WALK_PENDING → IDLE
    │
    ├─ onStartCommand()
    │       └─ scheduleNextAlarm(type=EXERCISE, delayMins=20)
    │
    ├─ AlarmReceiver.onReceive()  [BroadcastReceiver]
    │       ├─ type == EXERCISE:
    │       │       └─ coroutine → pickRandom(set1) + pickRandom(set2) from Room
    │       │       └─ startActivity(ExerciseAlarmActivity)
    │       │       └─ setState(EXERCISE_PENDING)
    │       │
    │       └─ type == WALK:
    │               └─ startActivity(WalkAlarmActivity)
    │               └─ setState(WALK_PENDING)
    │
    ├─ onTaskComplete(type)
    │       ├─ type == EXERCISE → scheduleNextAlarm(type=WALK, delayMins=40)
    │       └─ type == WALK    → scheduleNextAlarm(type=EXERCISE, delayMins=20)
    │
    ├─ onSnooze()
    │       └─ scheduleNextAlarm(same type, delayMins=2)
    │
    ├─ onRestMode(durationMins)
    │       └─ cancelPendingAlarms()
    │       └─ scheduleRestEndAlarm(durationMins)
    │       └─ onRestEnd() → scheduleNextAlarm(type=EXERCISE, delayMins=20)
    │
    └─ BootReceiver.onReceive(BOOT_COMPLETED)
            └─ Restart AlarmCycleService if was active before reboot
```

### Key Coroutine Patterns Used

```kotlin
// AlarmReceiver.kt — picking exercises without blocking main thread
class AlarmReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        val pendingResult = goAsync()           // keep receiver alive while coroutine runs
        CoroutineScope(Dispatchers.IO).launch {
            val set1 = db.exerciseDao().getEnabledBySet("SET_1").random()
            val set2 = db.exerciseDao().getEnabledBySet("SET_2").random()
            withContext(Dispatchers.Main) {
                launchExerciseAlarm(context, set1, set2)
            }
            pendingResult.finish()
        }
    }
}

// HomeViewModel.kt — reactive progress data
class HomeViewModel(private val repo: ProgressRepository) : ViewModel() {
    val todayStats: StateFlow<DayStats> = repo.getTodayStats()
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(), DayStats())
}

// ExerciseRepository.kt — offline-first sync
suspend fun syncIfNeeded() {
    val remoteVersion = api.getVersion().version
    val localVersion  = db.cacheMetaDao().getVersion()
    if (remoteVersion > localVersion) {
        val exercises = api.getExercises().sets
        db.exerciseDao().replaceAll(exercises)
        db.cacheMetaDao().setVersion(remoteVersion)
    }
}
```

---

## 14. Admin-Configurable Exercise System

### 14.1 How It Works

```
Admin Panel (Web)
      │
      └── REST API (JSON)
              │
              ├── GET /exercises          ← full exercise list with metadata
              └── GET /exercises/version  ← lightweight version check

App on Launch:
      1. Check local cache version vs remote version
      2. If changed → fetch full list → store in Room DB
      3. Display exercises from local DB (always offline-first)
```

### 14.2 Exercise Data Model (JSON from API)

```json
{
  "version": 4,
  "sets": [
    {
      "set_id": "SET_1",
      "set_name": "Lower Body & Core Pump",
      "exercises": [
        {
          "id": "EX_001",
          "name": "Slow Standing March",
          "duration_type": "TIME",
          "duration_value": 30,
          "duration_unit": "seconds",
          "sides": false,
          "coaching_cue": "Lift knees just high enough to engage hips — do NOT let the lower back round.",
          "video_url": "https://cdn.backhealth.app/3d/standing-march.mp4",
          "thumbnail_url": "https://cdn.backhealth.app/thumbs/standing-march.png",
          "enabled": true
        }
      ]
    }
  ]
}
```

### 14.3 Admin Panel Capabilities

| Feature | Description |
|---|---|
| Add exercise | Name, set assignment, duration, coaching cues, 3D video loop MP4 upload |
| Edit exercise | Update any field without app release |
| Disable exercise | Toggle off — removed from rotation immediately on next app sync |
| Reorder exercises | Drag to change display order in the library screen |
| Assign to set | Move exercise between Set 1 / Set 2 |
| Analytics view | See which exercises users mark as "too painful" most often |

> [!NOTE]
> Admin panel is a **Phase 1.5** deliverable — for the initial MVP, the exercise list is bundled as a local JSON file in the app assets and read into Room on first launch. The API sync layer is built in from the start so the switch to a live backend requires no structural change.

---

## 15. Database Schema (Room)

### `exercise` table (synced from admin)

| Column | Type | Description |
|---|---|---|
| `id` | TEXT PK | e.g., `EX_001` |
| `set_id` | TEXT | `SET_1` or `SET_2` |
| `name` | TEXT | Exercise name |
| `duration_type` | TEXT | `TIME` or `REPS` |
| `duration_value` | INTEGER | e.g., 30 or 10 |
| `duration_unit` | TEXT | `seconds`, `reps`, `reps_per_side` |
| `sides` | INTEGER | 1 = bilateral, 0 = single |
| `coaching_cue` | TEXT | Full instruction text |
| `video_url` | TEXT | Remote 3D animation loop MP4 URL |
| `local_video_res` | TEXT | Optional bundled raw resource identifier |
| `enabled` | INTEGER | 1 = active, 0 = disabled by admin |
| `sort_order` | INTEGER | Display order |

### `exercise_exclusion` table (user preferences)

| Column | Type | Description |
|---|---|---|
| `exercise_id` | TEXT FK | References `exercise.id` |
| `reason` | TEXT | `TOO_PAINFUL` / `NOT_SUITABLE` |
| `excluded_at` | INTEGER | Unix epoch ms |

### `exercise_log` table

| Column | Type | Description |
|---|---|---|
| `id` | INTEGER PK | Auto-increment |
| `date` | TEXT | YYYY-MM-DD |
| `exercise_id` | TEXT FK | References `exercise.id` |
| `exercise_name` | TEXT | Denormalised for history display |
| `set_id` | TEXT | `SET_1` or `SET_2` |
| `rounds_completed` | INTEGER | How many loops in the 2–3 min window |
| `completed` | INTEGER | 1=done, 0=snoozed/dismissed |
| `timestamp` | INTEGER | Unix epoch ms |

### `walk_log` table

| Column | Type | Description |
|---|---|---|
| `id` | INTEGER PK | Auto-increment |
| `date` | TEXT | YYYY-MM-DD |
| `completed` | INTEGER | 1=done, 0=snoozed |
| `timestamp` | INTEGER | Unix epoch ms |

### `user_prefs` table (single row)

| Column | Type |
|---|---|
| `user_type` | TEXT (`DISC_PATIENT` / `PREVENTIVE`) |
| `wake_time` | TEXT (HH:mm) |
| `bed_time` | TEXT (HH:mm) |

### `exercise_cache_meta` table (single row)

| Column | Type | Description |
|---|---|---|
| `version` | INTEGER | Last synced exercise list version |
| `last_synced_at` | INTEGER | Unix epoch ms |

---

## 15. Progress & Streak Tracking

| Metric | Tracked |
|---|---|
| Exercises completed per day | ✅ |
| Walks completed per day | ✅ |
| Snoozed count per day | ✅ |
| Daily streak (at least 3 exercises done) | ✅ |
| Weekly summary card | ✅ |

> [!NOTE]
> All data stored **locally** (Room database). No cloud sync in Phase 1.

---

## 16. Phase Roadmap

| Phase | Features | Stack |
|---|---|---|
| **Phase 1** | Lumbar focus · 20/40-min full-screen alarms · Exercise loops (GIF + text) · Walk breaks · Snooze · Rest mode · Sleep window · Progress tracking | Kotlin Android · Room · AlarmManager |
| **Phase 1.5** | Admin-configurable exercise library · Live GIF/content updates without app release · Basic analytics | + Spring Boot API · React Admin Panel |
| **Phase 2** | Cervical (neck) exercises · User-type exercise intensity · Hilt DI | Kotlin Android · Spring Boot |
| **Phase 3** | Auto walk detection via step counter · Personalised plans by disc level (L4-L5 / L5-S1) | Kotlin Android |
| **Phase 4** | Wearable integration · Cloud user sync · Doctor-shared routines | All layers |

---

## 18. Project Structure (Kotlin Android)

```
app/
├── src/main/
│   ├── java/com/backhealth/          ← Kotlin files live here too (.kt)
│   │   ├── alarm/
│   │   │   ├── AlarmCycleService.kt      ← Foreground service (state machine)
│   │   │   ├── AlarmReceiver.kt          ← BroadcastReceiver + goAsync() coroutine
│   │   │   └── BootReceiver.kt           ← Restart cycle after phone reboot
│   │   ├── ui/
│   │   │   ├── home/
│   │   │   │   ├── HomeActivity.kt
│   │   │   │   └── HomeViewModel.kt      ← StateFlow for reactive countdown + stats
│   │   │   ├── alarm/
│   │   │   │   ├── ExerciseAlarmActivity.kt  ← Full-screen exercise alarm
│   │   │   │   └── WalkAlarmActivity.kt      ← Full-screen walk alarm
│   │   │   ├── onboarding/
│   │   │   │   └── OnboardingActivity.kt
│   │   │   ├── library/
│   │   │   │   ├── ExerciseLibraryActivity.kt
│   │   │   │   └── ExerciseLibraryViewModel.kt
│   │   │   ├── progress/
│   │   │   │   └── ProgressActivity.kt
│   │   │   └── settings/
│   │   │       └── SettingsActivity.kt
│   │   ├── data/
│   │   │   ├── db/
│   │   │   │   ├── AppDatabase.kt
│   │   │   │   ├── ExerciseDao.kt            ← suspend fun queries (coroutine-native)
│   │   │   │   ├── ExerciseExclusionDao.kt
│   │   │   │   ├── ExerciseLogDao.kt
│   │   │   │   ├── WalkLogDao.kt
│   │   │   │   └── UserPrefsDao.kt
│   │   │   ├── model/
│   │   │   │   ├── Exercise.kt               ← data class (1 line each)
│   │   │   │   ├── ExerciseSet.kt
│   │   │   │   ├── ExerciseLog.kt
│   │   │   │   ├── ExerciseExclusion.kt
│   │   │   │   └── WalkLog.kt
│   │   │   ├── api/
│   │   │   │   ├── ExerciseApiService.kt     ← Retrofit suspend interface
│   │   │   │   └── ExerciseSyncManager.kt    ← Version check + sync coroutine
│   │   │   └── repository/
│   │   │       ├── ExerciseRepository.kt
│   │   │       └── ProgressRepository.kt
│   │   └── utils/
│   │       ├── AlarmScheduler.kt
│   │       ├── SleepWindowChecker.kt
│   │       └── ExercisePicker.kt             ← list.random() — 1 line in Kotlin
│   ├── assets/
│   │   └── exercises_default.json            ← Bundled exercise list for MVP
│   ├── res/
│   │   ├── raw/                              ← Bundled exercise demo loop videos (MP4/WebM)
│   │   ├── layout/                           ← XML layouts
│   │   └── values/                           ← Colors, strings, themes
│   └── AndroidManifest.xml
└── build.gradle.kts                          ← Kotlin DSL build script
```

---

*Document Version: 1.5 | Updated: 2026-11-16 | Status: Ready for Implementation (Looping Video Form Demos)*
