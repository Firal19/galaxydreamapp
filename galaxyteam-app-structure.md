# Galaxy Team Internal App - Expo & Supabase Architecture

## Technology Stack

### Frontend (Mobile App)
- **Framework**: Expo SDK 49+
- **Language**: TypeScript
- **State Management**: Zustand (lightweight alternative to Redux)
- **Navigation**: Expo Router (file-based routing)
- **UI Components**: Custom components + React Native Elements
- **Forms**: React Hook Form
- **Notifications**: Expo Notifications
- **Analytics**: Expo Analytics

### Backend (Supabase)
- **Database**: PostgreSQL (managed by Supabase)
- **Authentication**: Supabase Auth
- **Real-time**: Supabase Realtime subscriptions
- **Storage**: Supabase Storage (for profile pictures)
- **Edge Functions**: For ERP sync (if needed)
- **Row Level Security**: For data access control

## Project Structure

```
galaxy-team-app/
│
├── app/                            # Expo Router app directory
│   ├── (auth)/                     # Auth routes group
│   │   ├── login.tsx              # Login screen
│   │   ├── register.tsx           # Registration screen
│   │   └── _layout.tsx            # Auth layout
│   │
│   ├── (tabs)/                     # Main app tabs
│   │   ├── index.tsx              # Home/Dashboard
│   │   ├── content.tsx            # Content hub
│   │   ├── grade.tsx              # Grading screen
│   │   ├── analytics.tsx          # Personal analytics
│   │   ├── messages.tsx           # Communication
│   │   └── _layout.tsx            # Tab layout
│   │
│   ├── content/
│   │   ├── [id].tsx               # Content detail
│   │   └── create.tsx             # Submit for grading
│   │
│   ├── _layout.tsx                # Root layout
│   └── +not-found.tsx             # 404 screen
│
├── components/                     # Reusable components
│   ├── ui/                        # Basic UI components
│   │   ├── Button.tsx
│   │   ├── Card.tsx
│   │   ├── Input.tsx
│   │   ├── Badge.tsx
│   │   └── LoadingSpinner.tsx
│   │
│   ├── content/                   # Content-related components
│   │   ├── ContentCard.tsx
│   │   ├── GradeChecker.tsx
│   │   ├── ChecklistItem.tsx
│   │   └── TrackingLink.tsx
│   │
│   ├── analytics/                 # Analytics components
│   │   ├── ClickCounter.tsx
│   │   ├── DailyProgress.tsx
│   │   ├── SimpleChart.tsx
│   │   └── LeadNotification.tsx
│   │
│   └── common/                    # Shared components
│       ├── Header.tsx
│       ├── EmptyState.tsx
│       ├── ErrorBoundary.tsx
│       └── OfflineNotice.tsx
│
├── lib/                           # Core functionality
│   ├── supabase.ts               # Supabase client setup
│   ├── store.ts                  # Zustand store
│   ├── notifications.ts          # Push notification setup
│   └── analytics.ts              # Simple analytics helpers
│
├── utils/                         # Utility functions
│   ├── content-validator.ts      # Simple grading logic
│   ├── formatters.ts             # Date, number formatting
│   ├── constants.ts              # App constants
│   └── storage.ts                # Async storage helpers
│
├── hooks/                         # Custom React hooks
│   ├── useAuth.ts                # Authentication hook
│   ├── useContent.ts             # Content data hook
│   ├── useAnalytics.ts           # Analytics hook
│   ├── useSupabase.ts            # Supabase helpers
│   └── useNotifications.ts       # Notification hook
│
├── types/                         # TypeScript types
│   ├── database.types.ts         # Generated from Supabase
│   ├── content.types.ts          # Content-related types
│   └── index.ts                  # Common types
│
├── assets/                        # Static assets
│   ├── images/
│   ├── fonts/
│   └── icons/
│
├── styles/                        # Global styles
│   ├── colors.ts                 # Color palette
│   ├── typography.ts             # Font styles
│   └── spacing.ts                # Spacing system
│
├── supabase/                      # Supabase configuration
│   ├── migrations/               # Database migrations
│   │   ├── 001_initial_schema.sql
│   │   ├── 002_content_tables.sql
│   │   └── 003_analytics_tables.sql
│   │
│   ├── functions/                # Edge functions (if needed)
│   │   └── sync-with-erp/
│   │       └── index.ts
│   │
│   └── seed.sql                  # Sample data
│
├── app.json                       # Expo configuration
├── babel.config.js               # Babel configuration
├── tsconfig.json                 # TypeScript config
├── package.json                  # Dependencies
├── .env.example                  # Environment variables
└── README.md                     # Documentation
```

## Supabase Database Schema

```sql
-- Users table (extends Supabase auth.users)
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id),
  full_name TEXT NOT NULL,
  phone_number TEXT UNIQUE NOT NULL,
  office_id UUID NOT NULL,
  upline_id UUID REFERENCES profiles(id),
  role TEXT DEFAULT 'member',
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Offices table
CREATE TABLE offices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  city TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Content ideas from leaders
CREATE TABLE content_ideas (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  leader_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  description TEXT NOT NULL,
  leader_comment TEXT,
  theme_week INTEGER NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Content submissions for grading
CREATE TABLE content_submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id),
  content_idea_id UUID REFERENCES content_ideas(id),
  content_text TEXT NOT NULL,
  has_headline BOOLEAN DEFAULT FALSE,
  has_cta BOOLEAN DEFAULT FALSE,
  has_body BOOLEAN DEFAULT FALSE,
  has_hashtags BOOLEAN DEFAULT FALSE,
  passed BOOLEAN DEFAULT FALSE,
  tracking_id TEXT UNIQUE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Simple analytics
CREATE TABLE link_analytics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tracking_id TEXT REFERENCES content_submissions(tracking_id),
  user_id UUID REFERENCES profiles(id),
  click_count INTEGER DEFAULT 0,
  last_clicked TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Lead notifications
CREATE TABLE lead_notifications (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id),
  tracking_id TEXT,
  clicked_at TIMESTAMPTZ DEFAULT NOW(),
  synced_to_erp BOOLEAN DEFAULT FALSE
);

-- Simple messages
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sender_id UUID REFERENCES profiles(id),
  recipient_id UUID REFERENCES profiles(id),
  message TEXT NOT NULL,
  read BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Announcements
CREATE TABLE announcements (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  author_id UUID REFERENCES profiles(id),
  title TEXT NOT NULL,
  content TEXT NOT NULL,
  target_office_id UUID REFERENCES offices(id),
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

## Key Implementation Files

### 1. Supabase Client Setup
```typescript
// lib/supabase.ts
import { createClient } from '@supabase/supabase-js'
import AsyncStorage from '@react-native-async-storage/async-storage'
import { Database } from '../types/database.types'

const supabaseUrl = process.env.EXPO_PUBLIC_SUPABASE_URL!
const supabaseAnonKey = process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY!

export const supabase = createClient<Database>(supabaseUrl, supabaseAnonKey, {
  auth: {
    storage: AsyncStorage,
    autoRefreshToken: true,
    persistSession: true,
    detectSessionInUrl: false,
  },
})
```

### 2. Simple State Management
```typescript
// lib/store.ts
import { create } from 'zustand'

interface AppState {
  user: Profile | null
  contentIdeas: ContentIdea[]
  analytics: Analytics | null
  setUser: (user: Profile | null) => void
  setContentIdeas: (ideas: ContentIdea[]) => void
  setAnalytics: (analytics: Analytics) => void
}

export const useStore = create<AppState>((set) => ({
  user: null,
  contentIdeas: [],
  analytics: null,
  setUser: (user) => set({ user }),
  setContentIdeas: (contentIdeas) => set({ contentIdeas }),
  setAnalytics: (analytics) => set({ analytics }),
}))
```

### 3. Content Validator
```typescript
// utils/content-validator.ts
export const validateContent = (content: string) => {
  const lines = content.split('\n')
  const words = content.split(' ').filter(word => word.length > 0)
  const hashtags = content.match(/#\w+/g) || []
  
  return {
    hasHeadline: lines[0]?.split(' ').length >= 5 && lines[0]?.split(' ').length <= 15,
    hasCta: /contact|call|whatsapp|message|dm|link|click/i.test(content),
    hasBody: words.length >= 50 && words.length <= 300,
    hasHashtags: hashtags.length >= 3 && hashtags.some(tag => tag.toLowerCase() === '#galaxyteam'),
    passed: false // Set after all checks
  }
}
```

## Development Workflow

### Local Development with Expo
```bash
# Install dependencies
npm install

# Start Expo development server
npx expo start

# Run on iOS simulator
npx expo run:ios

# Run on Android emulator
npx expo run:android
```

### Supabase Local Development
```bash
# Start Supabase locally
npx supabase start

# Run migrations
npx supabase db push

# Generate TypeScript types
npx supabase gen types typescript --local > types/database.types.ts
```

## Build & Deployment

### Building with EAS Build
```bash
# Install EAS CLI
npm install -g eas-cli

# Configure EAS
eas build:configure

# Build for iOS
eas build --platform ios

# Build for Android
eas build --platform android
```

### Environment Configuration
```javascript
// app.json
{
  "expo": {
    "name": "Galaxy Team",
    "slug": "galaxy-team",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "automatic",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#0B5CFF"
    },
    "ios": {
      "supportsTablet": false,
      "bundleIdentifier": "com.galaxyteam.app"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#0B5CFF"
      },
      "package": "com.galaxyteam.app"
    },
    "extra": {
      "eas": {
        "projectId": "your-eas-project-id"
      }
    }
  }
}
```

## Integration Points

### ERP Sync Options

1. **Webhook from Supabase**
   - Trigger on new leads
   - Send to ERP endpoint
   - Mark as synced

2. **Scheduled Edge Function**
   - Run every 5 minutes
   - Batch sync data
   - Update sync status

3. **Direct API from App**
   - Call ERP API directly
   - Handle in background
   - Retry on failure

This simplified structure focuses on essential features while maintaining code quality and scalability. The architecture leverages Expo and Supabase's built-in capabilities, reducing complexity while delivering a robust solution.