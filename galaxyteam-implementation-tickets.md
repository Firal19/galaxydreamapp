# Galaxy Team Internal App - Detailed Implementation Tickets

## Project Foundation (Week 1)

### TICKET-001: Expo Project Initialization
**Priority**: Critical  
**Estimated Hours**: 4  
**Assignee**: Senior Developer

**Detailed Tasks**:
```bash
# Step 1: Create new Expo project with TypeScript
npx create-expo-app galaxy-team-app --template expo-template-blank-typescript
cd galaxy-team-app

# Step 2: Install core dependencies
npm install @supabase/supabase-js @react-native-async-storage/async-storage
npm install zustand react-hook-form expo-router expo-notifications
npm install react-native-elements react-native-vector-icons
npm install react-native-safe-area-context react-native-screens

# Step 3: Install dev dependencies
npm install -D @types/react @types/react-native eslint prettier
```

**Configuration Files**:
- [ ] Create `tsconfig.json` with strict mode enabled
- [ ] Setup `.eslintrc.js` with React Native rules
- [ ] Configure `.prettierrc` for consistent formatting
- [ ] Create `app.json` with proper app metadata
- [ ] Setup `babel.config.js` for Expo Router
- [ ] Create `.gitignore` with proper exclusions
- [ ] Setup `.env.example` with required variables:
  ```
  EXPO_PUBLIC_SUPABASE_URL=
  EXPO_PUBLIC_SUPABASE_ANON_KEY=
  EXPO_PUBLIC_ERP_WEBHOOK_URL=
  ```

**Folder Structure Creation**:
- [ ] Create all directories as per architecture document
- [ ] Add placeholder files in each folder
- [ ] Setup assets folder with placeholder images
- [ ] Create README.md with setup instructions

**Acceptance Criteria**:
- Project runs on iOS simulator
- Project runs on Android emulator
- TypeScript compilation works
- Linting passes without errors

### TICKET-002: Supabase Project Setup
**Priority**: Critical  
**Estimated Hours**: 6  
**Assignee**: Backend Developer

**Supabase Dashboard Tasks**:
1. Create new Supabase project in dashboard
2. Note down project URL and anon key
3. Enable phone auth in Authentication settings
4. Configure SMS provider (Twilio/MessageBird)
5. Set up allowed phone number patterns for Ethiopia (+251)

**Database Schema Creation**:
```sql
-- Run these migrations in order in Supabase SQL editor

-- Migration 001: Core tables
CREATE TABLE offices (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  city TEXT NOT NULL,
  manager_name TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Insert 20 offices
INSERT INTO offices (name, city) VALUES
  ('Galaxy HQ', 'Addis Ababa'),
  ('Unity Center', 'Addis Ababa'),
  ('Vision Tower', 'Bahir Dar'),
  ('Progress Plaza', 'Dire Dawa'),
  ('Summit Hub', 'Mekelle'),
  ('Nile Tower', 'Hawassa'),
  ('Skyline Plaza', 'Adama'),
  ('Ethiopia Tower', 'Gondar'),
  ('Renaissance Plaza', 'Jimma'),
  ('Victory Center', 'Dessie'),
  ('Success Hub', 'Nazret'),
  ('Achievement Plaza', 'Bishoftu'),
  ('Growth Center', 'Debre Zeit'),
  ('Prosperity Tower', 'Sodo'),
  ('Excellence Hub', 'Arba Minch'),
  ('Innovation Plaza', 'Debre Markos'),
  ('Future Center', 'Harar'),
  ('Rising Tower', 'Shashemene'),
  ('Unity Hub', 'Woldia'),
  ('Dream Plaza', 'Aksum');

-- Create profiles table
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  full_name TEXT NOT NULL,
  phone_number TEXT UNIQUE NOT NULL,
  office_id UUID NOT NULL REFERENCES offices(id),
  upline_id UUID REFERENCES profiles(id),
  role TEXT DEFAULT 'member' CHECK (role IN ('member', 'leader', 'admin')),
  tracking_base_url TEXT UNIQUE,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE offices ENABLE ROW LEVEL SECURITY;

-- Create RLS policies
CREATE POLICY "Users can view their own profile" ON profiles
  FOR SELECT USING (auth.uid() = id);

CREATE POLICY "Users can update their own profile" ON profiles
  FOR UPDATE USING (auth.uid() = id);

CREATE POLICY "Everyone can view offices" ON offices
  FOR SELECT USING (true);
```

**Storage Buckets**:
- [ ] Create 'avatars' bucket for profile pictures
- [ ] Set bucket to public
- [ ] Configure size limits (max 5MB)
- [ ] Set allowed file types (jpg, png)

**Edge Functions Setup** (if needed for ERP sync):
```typescript
// supabase/functions/sync-to-erp/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'

serve(async (req) => {
  const { tracking_id, user_id, click_count } = await req.json()
  
  // Send to ERP
  const response = await fetch(Deno.env.get('ERP_WEBHOOK_URL'), {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ tracking_id, user_id, click_count })
  })
  
  return new Response(JSON.stringify({ success: true }))
})
```

**Acceptance Criteria**:
- All tables created successfully
- RLS policies working
- Can insert test data
- Phone auth configured

### TICKET-003: Authentication Flow Implementation
**Priority**: Critical  
**Estimated Hours**: 12  
**Assignee**: Senior Developer

**File: `app/(auth)/login.tsx`**
```typescript
import { useState } from 'react'
import { View, Text, TextInput, Alert } from 'react-native'
import { Button } from 'react-native-elements'
import { supabase } from '../../lib/supabase'
import { router } from 'expo-router'

export default function LoginScreen() {
  const [phone, setPhone] = useState('')
  const [otp, setOtp] = useState('')
  const [loading, setLoading] = useState(false)
  const [otpSent, setOtpSent] = useState(false)

  const sendOTP = async () => {
    setLoading(true)
    // Format phone number for Ethiopia
    const formattedPhone = phone.startsWith('+251') ? phone : `+251${phone}`
    
    const { error } = await supabase.auth.signInWithOtp({
      phone: formattedPhone,
    })
    
    if (error) {
      Alert.alert('Error', error.message)
    } else {
      setOtpSent(true)
      Alert.alert('Success', 'OTP sent to your phone')
    }
    setLoading(false)
  }

  const verifyOTP = async () => {
    setLoading(true)
    const formattedPhone = phone.startsWith('+251') ? phone : `+251${phone}`
    
    const { error } = await supabase.auth.verifyOtp({
      phone: formattedPhone,
      token: otp,
      type: 'sms'
    })
    
    if (error) {
      Alert.alert('Error', error.message)
    } else {
      router.replace('/(tabs)')
    }
    setLoading(false)
  }

  return (
    <View style={styles.container}>
      {/* Implementation details */}
    </View>
  )
}
```

**File: `app/(auth)/register.tsx`**
- [ ] Create registration form with:
  - Full name input
  - Phone number input with +251 prefix
  - Office selection dropdown (20 offices)
  - Upline phone number input (optional)
  - Terms acceptance checkbox
- [ ] Implement form validation
- [ ] Handle registration flow
- [ ] Create profile after auth success
- [ ] Navigate to onboarding

**File: `hooks/useAuth.ts`**
```typescript
import { useEffect, useState } from 'react'
import { supabase } from '../lib/supabase'
import { useStore } from '../lib/store'

export function useAuth() {
  const [loading, setLoading] = useState(true)
  const { user, setUser } = useStore()

  useEffect(() => {
    // Check current session
    supabase.auth.getSession().then(({ data: { session } }) => {
      if (session?.user) {
        fetchProfile(session.user.id)
      }
      setLoading(false)
    })

    // Listen for auth changes
    const { data: { subscription } } = supabase.auth.onAuthStateChange(
      async (event, session) => {
        if (session?.user) {
          await fetchProfile(session.user.id)
        } else {
          setUser(null)
        }
      }
    )

    return () => subscription.unsubscribe()
  }, [])

  const fetchProfile = async (userId: string) => {
    const { data, error } = await supabase
      .from('profiles')
      .select('*, office:offices(*)')
      .eq('id', userId)
      .single()
    
    if (data) setUser(data)
  }

  return { user, loading }
}
```

**Acceptance Criteria**:
- Can register new users
- OTP verification works
- Profile created after registration
- Session persists after app restart
- Proper error handling

## Core Features Implementation (Week 2-3)

### TICKET-004: Content Distribution System
**Priority**: High  
**Estimated Hours**: 16  
**Assignee**: Full-Stack Developer

**Database Schema**:
```sql
-- Content ideas table
CREATE TABLE content_ideas (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  leader_id UUID REFERENCES profiles(id) NOT NULL,
  title TEXT NOT NULL,
  description TEXT NOT NULL,
  leader_comment TEXT,
  theme_week INTEGER NOT NULL,
  active BOOLEAN DEFAULT true,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Who can see which content
CREATE TABLE content_visibility (
  content_idea_id UUID REFERENCES content_ideas(id),
  user_id UUID REFERENCES profiles(id),
  viewed_at TIMESTAMPTZ,
  PRIMARY KEY (content_idea_id, user_id)
);

-- RLS policies
CREATE POLICY "Leaders can create content" ON content_ideas
  FOR INSERT WITH CHECK (
    EXISTS (
      SELECT 1 FROM profiles 
      WHERE id = auth.uid() 
      AND role IN ('leader', 'admin')
    )
  );

CREATE POLICY "Users see content for them" ON content_ideas
  FOR SELECT USING (
    EXISTS (
      SELECT 1 FROM content_visibility 
      WHERE content_idea_id = id 
      AND user_id = auth.uid()
    )
  );
```

**File: `app/(tabs)/content.tsx`**
```typescript
export default function ContentHub() {
  const [contentIdeas, setContentIdeas] = useState<ContentIdea[]>([])
  const [loading, setLoading] = useState(true)
  const { user } = useAuth()

  useEffect(() => {
    fetchContentIdeas()
    
    // Real-time subscription
    const subscription = supabase
      .channel('content_changes')
      .on('postgres_changes', {
        event: 'INSERT',
        schema: 'public',
        table: 'content_ideas'
      }, handleNewContent)
      .subscribe()

    return () => {
      subscription.unsubscribe()
    }
  }, [])

  const fetchContentIdeas = async () => {
    const { data, error } = await supabase
      .from('content_ideas')
      .select(`
        *,
        leader:leader_id(full_name)
      `)
      .order('created_at', { ascending: false })
      .limit(10)

    if (data) setContentIdeas(data)
    setLoading(false)
  }

  // Component renders list of ContentCard components
}
```

**Leader Content Creation**:
```typescript
// components/content/CreateContentIdea.tsx
export function CreateContentIdea() {
  const [title, setTitle] = useState('')
  const [description, setDescription] = useState('')
  const [comment, setComment] = useState('')
  
  const createIdea = async () => {
    // Get current theme week
    const themeWeek = getThemeWeek()
    
    // Create content idea
    const { data: idea, error } = await supabase
      .from('content_ideas')
      .insert({
        title,
        description,
        leader_comment: comment,
        theme_week: themeWeek,
        leader_id: user.id
      })
      .select()
      .single()

    if (idea) {
      // Add visibility for downline members
      await addVisibilityForDownline(idea.id)
      
      // Send push notifications
      await sendPushToDownline()
    }
  }
}
```

**Push Notifications**:
```typescript
// utils/notifications.ts
import * as Notifications from 'expo-notifications'

export async function sendPushToDownline(leaderId: string, title: string) {
  // Get downline members with push tokens
  const { data: downline } = await supabase
    .from('profiles')
    .select('id, push_token')
    .eq('upline_id', leaderId)
    .not('push_token', 'is', null)

  // Send notifications
  const messages = downline.map(member => ({
    to: member.push_token,
    title: 'New Content Idea!',
    body: title,
    data: { type: 'content_idea' }
  }))

  await Notifications.scheduleNotificationAsync({
    content: messages,
    trigger: null
  })
}
```

**Acceptance Criteria**:
- Leaders can create content ideas
- Members see relevant content
- Push notifications work
- Real-time updates display
- Content marked as viewed

### TICKET-005: Simple Content Grading System
**Priority**: High  
**Estimated Hours**: 10  
**Assignee**: Full-Stack Developer

**Database Schema**:
```sql
CREATE TABLE content_submissions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES profiles(id) NOT NULL,
  content_idea_id UUID REFERENCES content_ideas(id),
  content_text TEXT NOT NULL,
  has_headline BOOLEAN DEFAULT false,
  has_cta BOOLEAN DEFAULT false,
  has_body BOOLEAN DEFAULT false,
  has_hashtags BOOLEAN DEFAULT false,
  passed BOOLEAN DEFAULT false,
  tracking_id TEXT UNIQUE,
  attempt_number INTEGER DEFAULT 1,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Tracking IDs sequence
CREATE SEQUENCE tracking_id_seq START 10000;
```

**File: `utils/content-validator.ts`**
```typescript
export interface ValidationResult {
  hasHeadline: boolean
  hasCta: boolean
  hasBody: boolean
  hasHashtags: boolean
  passed: boolean
  feedback: string[]
}

export function validateContent(content: string): ValidationResult {
  const lines = content.trim().split('\n')
  const firstLine = lines[0] || ''
  const words = content.split(/\s+/).filter(word => word.length > 0)
  const hashtags = content.match(/#\w+/g) || []
  
  const result: ValidationResult = {
    hasHeadline: false,
    hasCta: false,
    hasBody: false,
    hasHashtags: false,
    passed: false,
    feedback: []
  }
  
  // Check 1: Headline (5-15 words in first line)
  const headlineWords = firstLine.split(/\s+/).filter(w => w.length > 0)
  if (headlineWords.length >= 5 && headlineWords.length <= 15) {
    result.hasHeadline = true
  } else {
    result.feedback.push(
      headlineWords.length < 5 
        ? 'Headline too short. Need at least 5 words.'
        : 'Headline too long. Maximum 15 words.'
    )
  }
  
  // Check 2: Call to Action
  const ctaKeywords = [
    'contact', 'call', 'whatsapp', 'message', 'dm', 
    'click', 'join', 'register', 'sign up', 'learn more',
    'ደውሉ', 'መልእክት', 'ይቀላቀሉ' // Amharic CTAs
  ]
  
  const hasCta = ctaKeywords.some(keyword => 
    content.toLowerCase().includes(keyword)
  )
  
  if (hasCta) {
    result.hasCta = true
  } else {
    result.feedback.push('Missing call to action. Add how people can contact you.')
  }
  
  // Check 3: Body (50-300 words)
  if (words.length >= 50 && words.length <= 300) {
    result.hasBody = true
  } else {
    result.feedback.push(
      words.length < 50
        ? `Too short. You have ${words.length} words, need at least 50.`
        : `Too long. You have ${words.length} words, maximum is 300.`
    )
  }
  
  // Check 4: Hashtags (minimum 3, must include #GalaxyTeam)
  const hasGalaxyTag = hashtags.some(tag => 
    tag.toLowerCase() === '#galaxyteam'
  )
  
  if (hashtags.length >= 3 && hasGalaxyTag) {
    result.hasHashtags = true
  } else {
    if (!hasGalaxyTag) {
      result.feedback.push('Must include #GalaxyTeam hashtag')
    }
    if (hashtags.length < 3) {
      result.feedback.push(`Need at least 3 hashtags. You have ${hashtags.length}.`)
    }
  }
  
  // Overall pass/fail
  result.passed = result.hasHeadline && result.hasCta && 
                  result.hasBody && result.hasHashtags
  
  return result
}
```

**File: `app/(tabs)/grade.tsx`**
```typescript
export default function GradeScreen() {
  const [content, setContent] = useState('')
  const [validation, setValidation] = useState<ValidationResult | null>(null)
  const [submitting, setSubmitting] = useState(false)
  const { user } = useAuth()

  const checkContent = () => {
    const result = validateContent(content)
    setValidation(result)
  }

  const submitContent = async () => {
    if (!validation?.passed) return
    
    setSubmitting(true)
    
    // Generate tracking ID
    const trackingId = await generateTrackingId()
    
    // Save submission
    const { error } = await supabase
      .from('content_submissions')
      .insert({
        user_id: user.id,
        content_text: content,
        has_headline: validation.hasHeadline,
        has_cta: validation.hasCta,
        has_body: validation.hasBody,
        has_hashtags: validation.hasHashtags,
        passed: true,
        tracking_id: trackingId
      })
    
    if (!error) {
      // Generate tracking link
      const trackingLink = `https://galaxydreamteam.com/t/${trackingId}`
      
      // Show success with copy button
      Alert.alert(
        'Success!',
        `Your tracking link: ${trackingLink}`,
        [
          { text: 'Copy Link', onPress: () => copyToClipboard(trackingLink) },
          { text: 'OK' }
        ]
      )
      
      // Reset form
      setContent('')
      setValidation(null)
    }
    
    setSubmitting(false)
  }

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Grade Your Content</Text>
      
      <TextInput
        style={styles.textArea}
        multiline
        numberOfLines={10}
        value={content}
        onChangeText={setContent}
        placeholder="Paste your content here..."
        onBlur={checkContent}
      />
      
      {validation && (
        <View style={styles.results}>
          <ChecklistItem 
            label="Headline (5-15 words)" 
            checked={validation.hasHeadline} 
          />
          <ChecklistItem 
            label="Call to Action" 
            checked={validation.hasCta} 
          />
          <ChecklistItem 
            label="Body (50-300 words)" 
            checked={validation.hasBody} 
          />
          <ChecklistItem 
            label="3+ Hashtags (#GalaxyTeam)" 
            checked={validation.hasHashtags} 
          />
          
          {validation.feedback.length > 0 && (
            <View style={styles.feedback}>
              {validation.feedback.map((item, index) => (
                <Text key={index} style={styles.feedbackText}>
                  • {item}
                </Text>
              ))}
            </View>
          )}
        </View>
      )}
      
      <Button
        title="Submit Content"
        onPress={submitContent}
        disabled={!validation?.passed || submitting}
        loading={submitting}
      />
    </ScrollView>
  )
}
```

**Acceptance Criteria**:
- Content validation works correctly
- Real-time feedback as user types
- Tracking ID generated for passed content
- Can copy tracking link
- Validation state persists during edits

### TICKET-006: Simple Analytics Display
**Priority**: High  
**Estimated Hours**: 8  
**Assignee**: Frontend Developer

**Database Schema**:
```sql
-- Link click tracking
CREATE TABLE link_clicks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  tracking_id TEXT NOT NULL,
  clicked_at TIMESTAMPTZ DEFAULT NOW(),
  ip_address INET,
  user_agent TEXT,
  referrer TEXT
);

-- Aggregated analytics (updated by trigger)
CREATE TABLE link_analytics (
  tracking_id TEXT PRIMARY KEY,
  user_id UUID REFERENCES profiles(id),
  total_clicks INTEGER DEFAULT 0,
  unique_clicks INTEGER DEFAULT 0,
  last_clicked TIMESTAMPTZ,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Function to update analytics
CREATE OR REPLACE FUNCTION update_link_analytics()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO link_analytics (tracking_id, total_clicks, last_clicked)
  VALUES (NEW.tracking_id, 1, NOW())
  ON CONFLICT (tracking_id)
  DO UPDATE SET 
    total_clicks = link_analytics.total_clicks + 1,
    last_clicked = NOW();
  
  -- Send notification if this creates a lead
  PERFORM pg_notify('new_lead', json_build_object(
    'tracking_id', NEW.tracking_id,
    'clicked_at', NEW.clicked_at
  )::text);
  
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_analytics_on_click
AFTER INSERT ON link_clicks
FOR EACH ROW
EXECUTE FUNCTION update_link_analytics();
```

**File: `app/(tabs)/analytics.tsx`**
```typescript
export default function AnalyticsScreen() {
  const [analytics, setAnalytics] = useState({
    todayClicks: 0,
    weekClicks: 0,
    monthClicks: 0,
    topPost: null,
    recentLeads: []
  })
  const { user } = useAuth()

  useEffect(() => {
    fetchAnalytics()
    
    // Subscribe to real-time updates
    const subscription = supabase
      .channel('analytics_updates')
      .on('postgres_changes', {
        event: 'UPDATE',
        schema: 'public',
        table: 'link_analytics',
        filter: `user_id=eq.${user.id}`
      }, handleAnalyticsUpdate)
      .subscribe()

    return () => subscription.unsubscribe()
  }, [])

  const fetchAnalytics = async () => {
    // Get today's clicks
    const today = new Date().toISOString().split('T')[0]
    const { data: todayData } = await supabase
      .from('link_analytics')
      .select('total_clicks')
      .eq('user_id', user.id)
      .gte('last_clicked', today)
    
    const todayClicks = todayData?.reduce((sum, item) => 
      sum + item.total_clicks, 0
    ) || 0

    // Get week's clicks
    const weekAgo = new Date()
    weekAgo.setDate(weekAgo.getDate() - 7)
    const { data: weekData } = await supabase
      .from('link_analytics')
      .select('total_clicks')
      .eq('user_id', user.id)
      .gte('last_clicked', weekAgo.toISOString())
    
    const weekClicks = weekData?.reduce((sum, item) => 
      sum + item.total_clicks, 0
    ) || 0

    // Get top performing post
    const { data: topPost } = await supabase
      .from('link_analytics')
      .select(`
        tracking_id,
        total_clicks,
        content_submissions!inner(content_text)
      `)
      .eq('user_id', user.id)
      .order('total_clicks', { ascending: false })
      .limit(1)
      .single()

    setAnalytics({
      todayClicks,
      weekClicks,
      monthClicks: 0, // Calculate similarly
      topPost,
      recentLeads: []
    })
  }

  return (
    <ScrollView style={styles.container}>
      <Text style={styles.title}>Your Analytics</Text>
      
      <View style={styles.metricsGrid}>
        <MetricCard
          title="Today"
          value={analytics.todayClicks}
          icon="trending-up"
        />
        <MetricCard
          title="This Week"
          value={analytics.weekClicks}
          icon="calendar"
        />
      </View>
      
      {analytics.topPost && (
        <Card style={styles.topPostCard}>
          <Text style={styles.cardTitle}>Best Performing Post</Text>
          <Text style={styles.postPreview}>
            {analytics.topPost.content_submissions.content_text.slice(0, 100)}...
          </Text>
          <Text style={styles.clickCount}>
            {analytics.topPost.total_clicks} clicks
          </Text>
        </Card>
      )}
      
      <TouchableOpacity 
        style={styles.erpButton}
        onPress={() => openERPDashboard()}
      >
        <Text>View Detailed Analytics in ERP →</Text>
      </TouchableOpacity>
    </ScrollView>
  )
}
```

**Lead Notifications**:
```typescript
// hooks/useLeadNotifications.ts
export function useLeadNotifications() {
  const { user } = useAuth()
  
  useEffect(() => {
    // Request notification permissions
    Notifications.requestPermissionsAsync()
    
    // Listen for new leads
    const subscription = supabase
      .channel('new_leads')
      .on('postgres_changes', {
        event: 'INSERT',
        schema: 'public',
        table: 'link_clicks'
      }, async (payload) => {
        // Check if this click belongs to user
        const { data } = await supabase
          .from('link_analytics')
          .select('user_id')
          .eq('tracking_id', payload.new.tracking_id)
          .single()
        
        if (data?.user_id === user.id) {
          // Send local notification
          await Notifications.scheduleNotificationAsync({
            content: {
              title: 'New Lead! 🎉',
              body: 'Someone clicked your link just now',
              data: { tracking_id: payload.new.tracking_id }
            },
            trigger: null
          })
        }
      })
      .subscribe()

    return () => subscription.unsubscribe()
  }, [user])
}
```

**Acceptance Criteria**:
- Shows today's clicks accurately
- Shows week's total clicks
- Displays top performing post
- Real-time updates work
- Lead notifications trigger

## Communication Features (Week 4)

### TICKET-007: Simple Messaging System
**Priority**: Medium  
**Estimated Hours**: 8  
**Assignee**: Frontend Developer

**Database Schema**:
```sql
CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  sender_id UUID REFERENCES profiles(id) NOT NULL,
  recipient_id UUID REFERENCES profiles(id) NOT NULL,
  message TEXT NOT NULL,
  read BOOLEAN DEFAULT false,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Indexes for performance
CREATE INDEX idx_messages_recipient ON messages(recipient_id, created_at DESC);
CREATE INDEX idx_messages_sender ON messages(sender_id, created_at DESC);

-- RLS policies
CREATE POLICY "Users can see their messages" ON messages
  FOR SELECT USING (
    auth.uid() IN (sender_id, recipient_id)
  );

CREATE POLICY "Users can send messages" ON messages
  FOR INSERT WITH CHECK (
    auth.uid() = sender_id
  );
```

**File: `app/(tabs)/messages.tsx`**
```typescript
export default function MessagesScreen() {
  const [conversations, setConversations] = useState([])
  const { user } = useAuth()

  useEffect(() => {
    fetchConversations()
    
    // Real-time messages
    const subscription = supabase
      .channel('messages')
      .on('postgres_changes', {
        event: 'INSERT',
        schema: 'public',
        table: 'messages',
        filter: `recipient_id=eq.${user.id}`
      }, handleNewMessage)
      .subscribe()

    return () => subscription.unsubscribe()
  }, [])

  const fetchConversations = async () => {
    // Get unique conversations
    const { data } = await supabase
      .from('messages')
      .select(`
        *,
        sender:sender_id(full_name, role),
        recipient:recipient_id(full_name, role)
      `)
      .or(`sender_id.eq.${user.id},recipient_id.eq.${user.id}`)
      .order('created_at', { ascending: false })
    
    // Group by conversation
    const grouped = groupByConversation(data)
    setConversations(grouped)
  }

  return (
    <View style={styles.container}>
      <FlatList
        data={conversations}
        renderItem={({ item }) => (
          <ConversationItem
            conversation={item}
            onPress={() => router.push(`/chat/${item.otherUser.id}`)}
          />
        )}
        keyExtractor={item => item.otherUser.id}
      />
      
      {user.upline_id && (
        <FAB
          icon="message"
          onPress={() => router.push(`/chat/${user.upline_id}`)}
          style={styles.fab}
        />
      )}
    </View>
  )
}
```

**Chat Screen**:
```typescript
// app/chat/[userId].tsx
export default function ChatScreen() {
  const { userId } = useLocalSearchParams()
  const [messages, setMessages] = useState([])
  const [newMessage, setNewMessage] = useState('')
  const { user } = useAuth()

  const sendMessage = async () => {
    if (!newMessage.trim()) return
    
    const { error } = await supabase
      .from('messages')
      .insert({
        sender_id: user.id,
        recipient_id: userId,
        message: newMessage.trim()
      })
    
    if (!error) {
      setNewMessage('')
      // Optimistically add message
      setMessages([...messages, {
        id: Date.now(),
        sender_id: user.id,
        message: newMessage.trim(),
        created_at: new Date().toISOString()
      }])
    }
  }

  return (
    <KeyboardAvoidingView style={styles.container}>
      <FlatList
        data={messages}
        inverted
        renderItem={({ item }) => (
          <MessageBubble
            message={item}
            isOwn={item.sender_id === user.id}
          />
        )}
      />
      
      <View style={styles.inputContainer}>
        <TextInput
          style={styles.input}
          value={newMessage}
          onChangeText={setNewMessage}
          placeholder="Type a message..."
          multiline
        />
        <Button
          icon="send"
          onPress={sendMessage}
          disabled={!newMessage.trim()}
        />
      </View>
    </KeyboardAvoidingView>
  )
}
```

**Acceptance Criteria**:
- Can message upline
- Can receive messages
- Real-time updates work
- Message read status updates
- Push notifications for new messages

### TICKET-008: Announcements System
**Priority**: Medium  
**Estimated Hours**: 6  
**Assignee**: Frontend Developer

**Implementation Details**:
```typescript
// Components for viewing announcements
export function AnnouncementsList() {
  const [announcements, setAnnouncements] = useState([])
  const { user } = useAuth()

  useEffect(() => {
    // Fetch announcements for user's office
    const fetchAnnouncements = async () => {
      const { data } = await supabase
        .from('announcements')
        .select(`
          *,
          author:author_id(full_name, role)
        `)
        .or(`target_office_id.eq.${user.office_id},target_office_id.is.null`)
        .order('created_at', { ascending: false })
        .limit(20)
      
      setAnnouncements(data || [])
    }

    fetchAnnouncements()
  }, [])

  return (
    <View>
      {announcements.map(announcement => (
        <AnnouncementCard
          key={announcement.id}
          title={announcement.title}
          content={announcement.content}
          author={announcement.author.full_name}
          timestamp={announcement.created_at}
        />
      ))}
    </View>
  )
}
```

## Final Polish & Deployment (Week 5)

### TICKET-009: Performance Optimization
**Priority**: High  
**Estimated Hours**: 8  
**Assignee**: Senior Developer

**Tasks**:
- [ ] Implement React.memo for list items
- [ ] Add image lazy loading
- [ ] Implement FlatList optimization
- [ ] Add skeleton loaders
- [ ] Optimize Supabase queries
- [ ] Implement offline support
- [ ] Add error boundaries
- [ ] Minimize bundle size

### TICKET-010: EAS Build Configuration
**Priority**: Critical  
**Estimated Hours**: 6  
**Assignee**: DevOps Engineer

**File: `eas.json`**
```json
{
  "cli": {
    "version": ">= 3.0.0"
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal"
    },
    "preview": {
      "distribution": "internal",
      "ios": {
        "simulator": true
      },
      "android": {
        "buildType": "apk"
      }
    },
    "production": {
      "ios": {
        "cocoapods": "1.11.3"
      },
      "android": {
        "buildType": "app-bundle"
      }
    }
  },
  "submit": {
    "production": {
      "ios": {
        "appleId": "your-apple-id",
        "ascAppId": "your-app-store-connect-id",
        "appleTeamId": "your-team-id"
      },
      "android": {
        "serviceAccountKeyPath": "./google-play-key.json",
        "track": "production"
      }
    }
  }
}
```

**Build Commands**:
```bash
# Development build
eas build --profile development --platform all

# Preview build for testing
eas build --profile preview --platform all

# Production build
eas build --profile production --platform all

# Submit to stores
eas submit --platform ios
eas submit --platform android
```

## Testing Requirements

### TICKET-011: Comprehensive Testing
**Priority**: High  
**Estimated Hours**: 12  
**Assignee**: QA Engineer

**Test Scenarios**:
1. **Authentication Flow**
   - Register with Ethiopian phone number
   - OTP verification
   - Login/logout
   - Session persistence

2. **Content Flow**
   - View content ideas
   - Submit content for grading
   - All 4 validation rules
   - Copy tracking link

3. **Analytics Flow**
   - View click counts
   - Real-time updates
   - Lead notifications

4. **Communication Flow**
   - Send message to upline
   - Receive messages
   - View announcements

5. **Edge Cases**
   - No internet connection
   - Invalid phone numbers
   - Empty states
   - Error handling

## Total Timeline: 5 Weeks

### Week 1: Foundation
- Project setup
- Supabase configuration
- Authentication

### Week 2-3: Core Features
- Content distribution
- Grading system
- Analytics

### Week 4: Communication
- Messaging
- Notifications
- Announcements

### Week 5: Polish & Deploy
- Performance optimization
- Testing
- Deployment

## Budget Estimate: $15,000 - $25,000

### Development Costs
- 1 Senior Developer (160 hours @ $100/hr): $16,000
- 1 Junior Developer (80 hours @ $50/hr): $4,000
- UI/UX Design: $2,000
- Testing & QA: $1,500

### Infrastructure Costs (Annual)
- Supabase Pro: $25/month = $300/year
- Expo EAS: $99/month = $1,188/year
- SMS costs (Twilio): ~$500/month
- Apple Developer Account: $99/year
- Google Play Account: $25 (one-time)

This simplified approach focuses on essential features while maintaining quality and scalability. The use of Expo and Supabase significantly reduces development time and complexity.