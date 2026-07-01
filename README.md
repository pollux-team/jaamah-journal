# Jamā'ah Tracker

100% offline, privacy-first Islamic habit tracker built with Expo SDK 57 and React Native. Track daily prayers, Sunnah habits, streaks, and analytics — all stored locally on device with SQLite.

## Features

- **Prayer Tracking** — Log status for Fajr, Dhuhr, Asr, Maghrib, and Isha (Jamāʾah / On-Time / Late / Missed)
- **Sunnah & Habit Stacking** — Track Sunnah prayers, Tahajjud, Morning/Evening Adhkar
- **Streak Engine** — Current and best streaks, excused-day logic, 30-day trend line
- **Analytics Dashboard** — GitHub-style heatmap, pie chart, per-prayer bar chart, trend line
- **Qibla Compass** — Device magnetometer-based Qibla direction
- **Prayer Times** — Offline calculation via `adhan` JS, 12 methods, Sunnah times
- **Prohibited Days Mode** — Day-level toggle (female users only)
- **Data Export / Import** — JSON backup via expo-sharing / expo-document-picker
- **Dark Mode** — Green Islamic brand theme, automatic light/dark support
- **4-Tab Navigation** — Today, Stats, Tools, Settings

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Expo SDK 57, React Native 0.86, React 19 |
| Routing | expo-router (file-based) |
| Database | expo-sqlite (SQLite, WAL mode) |
| Prayer Times | `adhan` JS (pure JS, offline) |
| Animations | react-native-reanimated 4.5 |
| Haptics | expo-haptics |
| Compass | expo-sensors (magnetometer) |
| Language | TypeScript 6 (strict) |
| Package Manager | bun |

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) 18+
- [bun](https://bun.sh/) (package manager)
- Expo CLI: `bunx expo`

### Install

```bash
bun install
```

### Run

```bash
bunx expo start        # dev server
bunx expo start --ios  # iOS simulator
bunx expo start --android  # Android emulator
```

### Lint

```bash
bunx expo lint
```

## Project Structure

```
jaamah-tracker/
├── app.json                  # Expo config (plugins, splash, adaptive icon)
├── tsconfig.json             # Strict TS, @/* path alias → ./src/*
├── src/
│   ├── app/                  # expo-router file-based routes
│   │   ├── _layout.tsx       # Root layout: DatabaseProvider + ThemeProvider
│   │   ├── index.tsx         # Today screen
│   │   ├── stats.tsx         # Stats screen
│   │   ├── tools.tsx         # Tools screen
│   │   └── settings.tsx      # Settings screen
│   ├── components/
│   │   ├── app-tabs.tsx      # Native tab bar (iOS/Android)
│   │   ├── app-tabs.web.tsx  # Web tab bar
│   │   ├── onboarding.tsx    # 5-step onboarding wizard
│   │   └── animated-icon.tsx # Splash overlay animation
│   ├── screens/
│   │   ├── Today.tsx         # Prayer tracker, habits, countdown
│   │   ├── Stats.tsx         # Heatmap, pie, bar chart, trend
│   │   ├── Tools.tsx         # Countdown, schedule, Qibla compass
│   │   └── Settings.tsx      # Export, gender, location, methods
│   ├── logic/
│   │   └── scorer.ts         # StreakEngine, scoring, metrics
│   ├── database/
│   │   └── sqlite.tsx        # Schema, migrations, CRUD, DatabaseProvider
│   ├── utils/
│   │   ├── prayer-times.ts   # adhan wrapper, 12 methods, SunnahTimes
│   │   ├── qibla.ts          # Magnetometer hook + Qibla direction
│   │   └── location.ts       # GPS permission + storage
│   ├── constants/
│   │   └── theme.ts          # Colors, Fonts, Spacing, AppTheme type
│   ├── hooks/
│   │   └── use-color-scheme.web.ts  # SSR hydration hook
│   └── types/
│       └── css.d.ts          # CSS module declarations
├── assets/                   # Icons, splash images
└── scripts/                  # reset-project.js
```

## Database Schema

SQLite with WAL mode, versioned migrations.

```sql
-- prayers table
CREATE TABLE prayers (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,          -- 'Fajr' | 'Dhuhr' | 'Asr' | 'Maghrib' | 'Isha'
  status TEXT,                 -- 'Jamāʾah' | 'On-Time' | 'Late' | 'Missed'
  date TEXT NOT NULL,          -- 'YYYY-MM-DD'
  created_at TEXT NOT NULL DEFAULT (datetime('now'))
);
CREATE UNIQUE INDEX idx_prayers_name_date ON prayers(name, date);

-- habits table
CREATE TABLE habits (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  key TEXT NOT NULL,           -- 'sunnah-fajr' | 'tahajjud' | 'adhkar-morning' | etc.
  completed INTEGER NOT NULL DEFAULT 0,  -- 0 | 1
  date TEXT NOT NULL,
  created_at TEXT NOT NULL DEFAULT (datetime('now'))
);
CREATE UNIQUE INDEX idx_habits_key_date ON habits(key, date);

-- settings table (key-value)
CREATE TABLE settings (
  key TEXT PRIMARY KEY,
  value TEXT NOT NULL
);
```

### Settings Keys

| Key | Type | Default | Description |
|-----|------|---------|-------------|
| `gender` | `'male' \| 'female'` | `'male'` | Enables prohibited days for female |
| `location` | `{ latitude, longitude }` | `null` | GPS coordinates for prayer times |
| `calcMethod` | `CalculationMethodKey` | `'MWL'` | Prayer time calculation method |
| `asrMethod` | `'Standard' \| 'Hanafi'` | `'Standard'` | Asr juristic method |
| `excusedMode` | `boolean` | `false` | Prohibited days toggle |

## Scoring System

| Status | Points |
|--------|--------|
| Jamāʾah | 3 |
| On-Time | 2 |
| Late / Qadha | 1 |
| Missed | 0 |

**Daily Score** = sum of prayer points (max 15).  
**Perfect Day** = all 5 prayers logged as Jamāʾah or On-Time.

### Streak Logic

- **Perfect day** → streak +1
- **Late / Missed** → streak resets to 0
- **Excused day** (prohibited days mode) → streak frozen (no increment, no reset)
- **No data logged today** → streak continues (pending)

## Theme

Green Islamic brand theme with automatic light/dark mode.

```typescript
Colors = {
  light: {
    primary: '#1a7a4c',    // deep green
    background: '#f8faf9',
    accent: '#c9a227',     // gold
    prohibited: '#9333ea', // purple
    female: '#db2777',     // pink
    // ...
  },
  dark: {
    primary: '#34d17a',    // bright green
    background: '#0c1210',
    accent: '#e8c547',
    prohibited: '#a855f7',
    female: '#f472b6',
    // ...
  }
}
```

## Onboarding Flow

5 steps for female users, 4 for male:

1. **Welcome** — App intro
2. **Gender** — Male (`figure.stand`) / Female (`figure.wave`)
3. **Location** — GPS permission for prayer times
4. **Calculation Method** — Pick from 12 methods + Asr method
5. **Prohibited Days** *(female only)* — Day-level toggle explanation

## Prayer Calculation Methods

| Key | Label |
|-----|-------|
| `MWL` | Muslim World League |
| `ISNA` | ISNA (North America) |
| `Egypt` | Egyptian General Authority |
| `Makkah` | Umm al-Qura, Makkah |
| `Karachi` | University of Karachi |
| `Dubai` | Dubai |
| `MoonsightingCommittee` | Moonsighting Committee |
| `Kuwait` | Kuwait |
| `Qatar` | Qatar |
| `Singapore` | Singapore |
| `Tehran` | Institute of Geophysics, Tehran |
| `Turkey` | Diyanet, Turkey |

## Key Dependencies

| Package | Purpose |
|---------|---------|
| `adhan` ^4.4.4 | Prayer time calculation (pure JS) |
| `expo-sqlite` ~57.0.0 | Local SQLite database |
| `expo-sensors` ~57.0.1 | Magnetometer for Qibla compass |
| `expo-haptics` ~57.0.0 | Tactile feedback |
| `expo-location` ~57.0.1 | GPS for prayer time coordinates |
| `expo-file-system` ~57.0.0 | Data export file I/O |
| `expo-sharing` ~57.0.1 | Native share sheet |
| `expo-document-picker` ~57.0.0 | Import backup files |
| `date-fns` ^4.4.0 | Date formatting and intervals |
| `react-native-reanimated` 4.5.0 | Animations |

## License

MIT
