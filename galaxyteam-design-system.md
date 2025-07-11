# Galaxy Team Internal App - Simplified Design System

## Design Philosophy

Simple, clean, and Ethiopian. Our design focuses on clarity and ease of use, ensuring every member can navigate the app effortlessly. We embrace minimalism while respecting cultural preferences for vibrant colors and community feeling.

## Color Palette

### Primary Colors
```scss
// Brand Colors
$primary: #0B5CFF;        // Galaxy Blue - Main brand color
$secondary: #00D4AA;      // Success Green - Money & success
$accent: #FFB800;         // Ethiopian Gold - Special highlights

// Semantic Colors
$success: #10B981;        // Pass/Success states
$error: #EF4444;          // Fail/Error states  
$warning: #F59E0B;        // Warning states
$info: #3B82F6;           // Information

// Grayscale
$black: #000000;          // Pure black
$gray-900: #111827;       // Darkest text
$gray-700: #374151;       // Secondary text
$gray-500: #6B7280;       // Disabled text
$gray-300: #D1D5DB;       // Borders
$gray-100: #F3F4F6;       // Backgrounds
$white: #FFFFFF;          // Pure white

// Dark Mode
$dark-bg: #1A1A2E;        // Dark background
$dark-surface: #252542;   // Dark cards
$dark-border: #373754;    // Dark borders
```

## Typography

### Font Family
```scss
// System fonts for performance
$font-primary: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
$font-ethiopic: 'Noto Sans Ethiopic', system-ui; // For Amharic text
```

### Font Sizes
```scss
// Simple scale
$text-xs: 12px;     // Captions, labels
$text-sm: 14px;     // Secondary text
$text-base: 16px;   // Body text
$text-lg: 18px;     // Subheadings
$text-xl: 20px;     // Headings
$text-2xl: 24px;    // Page titles
$text-3xl: 30px;    // Large numbers

// Font weights
$font-normal: 400;
$font-medium: 500;
$font-semibold: 600;
$font-bold: 700;
```

## Spacing

```scss
// 4px base unit
$space-1: 4px;
$space-2: 8px;
$space-3: 12px;
$space-4: 16px;
$space-5: 20px;
$space-6: 24px;
$space-8: 32px;
$space-10: 40px;
```

## Core Components

### 1. Buttons

#### Primary Button
```tsx
// Main actions
<Button 
  title="Share Content"
  onPress={handlePress}
  loading={loading}
/>

// Styles
const styles = StyleSheet.create({
  button: {
    backgroundColor: colors.primary,
    paddingVertical: 12,
    paddingHorizontal: 24,
    borderRadius: 8,
    alignItems: 'center',
  },
  buttonText: {
    color: colors.white,
    fontSize: 16,
    fontWeight: '600',
  },
  buttonPressed: {
    opacity: 0.8,
  }
})
```

#### Secondary Button
```tsx
// Secondary actions
<Button 
  title="Cancel"
  variant="secondary"
  onPress={handleCancel}
/>

// Styles
const secondaryButton = {
  backgroundColor: 'transparent',
  borderWidth: 1,
  borderColor: colors.primary,
}
```

### 2. Input Fields

#### Text Input
```tsx
<TextInput
  label="Phone Number"
  value={phone}
  onChangeText={setPhone}
  placeholder="+251..."
  keyboardType="phone-pad"
/>

// Styles
const styles = StyleSheet.create({
  inputContainer: {
    marginBottom: 16,
  },
  label: {
    fontSize: 14,
    color: colors.gray700,
    marginBottom: 8,
  },
  input: {
    borderWidth: 1,
    borderColor: colors.gray300,
    borderRadius: 8,
    paddingHorizontal: 16,
    paddingVertical: 12,
    fontSize: 16,
  },
  inputFocused: {
    borderColor: colors.primary,
    borderWidth: 2,
  }
})
```

#### Text Area
```tsx
<TextArea
  label="Your Content"
  value={content}
  onChangeText={setContent}
  placeholder="Write your post..."
  maxLength={300}
  showCounter
/>

// Additional styles
const textArea = {
  minHeight: 120,
  textAlignVertical: 'top',
}
```

### 3. Cards

#### Content Card
```tsx
<ContentCard
  title="Daily Motivation"
  description="Share an inspiring story"
  leader="Solomon G."
  timestamp="2 hours ago"
/>

// Styles
const styles = StyleSheet.create({
  card: {
    backgroundColor: colors.white,
    borderRadius: 12,
    padding: 16,
    marginBottom: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  cardTitle: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 8,
  },
  cardDescription: {
    fontSize: 14,
    color: colors.gray700,
    marginBottom: 12,
  },
  cardFooter: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
  }
})
```

#### Metric Card
```tsx
<MetricCard
  label="Today's Clicks"
  value={45}
  change="+12"
/>

// Styles
const styles = StyleSheet.create({
  metricCard: {
    backgroundColor: colors.primary,
    borderRadius: 12,
    padding: 16,
    flex: 1,
  },
  metricLabel: {
    color: colors.white,
    fontSize: 14,
    opacity: 0.9,
  },
  metricValue: {
    color: colors.white,
    fontSize: 30,
    fontWeight: 'bold',
    marginVertical: 4,
  },
  metricChange: {
    color: colors.secondary,
    fontSize: 14,
    fontWeight: '500',
  }
})
```

### 4. Checklist Items

#### Grade Checklist
```tsx
<ChecklistItem
  label="Has Headline (5-15 words)"
  checked={hasHeadline}
/>

// Styles
const styles = StyleSheet.create({
  checklistItem: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingVertical: 12,
    borderBottomWidth: 1,
    borderBottomColor: colors.gray100,
  },
  checkbox: {
    width: 24,
    height: 24,
    borderRadius: 12,
    borderWidth: 2,
    borderColor: colors.gray300,
    marginRight: 12,
    alignItems: 'center',
    justifyContent: 'center',
  },
  checkboxChecked: {
    backgroundColor: colors.success,
    borderColor: colors.success,
  },
  checkmark: {
    color: colors.white,
    fontSize: 16,
  },
  checklistLabel: {
    fontSize: 16,
    color: colors.gray900,
    flex: 1,
  }
})
```

### 5. Navigation

#### Tab Bar
```tsx
<Tab.Navigator
  screenOptions={{
    tabBarActiveTintColor: colors.primary,
    tabBarInactiveTintColor: colors.gray500,
    tabBarStyle: {
      borderTopWidth: 1,
      borderTopColor: colors.gray100,
    }
  }}
>
  <Tab.Screen name="Home" component={HomeScreen} />
  <Tab.Screen name="Content" component={ContentScreen} />
  <Tab.Screen name="Grade" component={GradeScreen} />
  <Tab.Screen name="Analytics" component={AnalyticsScreen} />
  <Tab.Screen name="Messages" component={MessagesScreen} />
</Tab.Navigator>
```

### 6. Feedback Components

#### Toast Notification
```tsx
<Toast
  message="Content graded successfully!"
  type="success"
  duration={3000}
/>

// Styles
const styles = StyleSheet.create({
  toast: {
    position: 'absolute',
    top: 50,
    left: 20,
    right: 20,
    backgroundColor: colors.white,
    borderRadius: 8,
    padding: 16,
    flexDirection: 'row',
    alignItems: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.1,
    shadowRadius: 8,
    elevation: 5,
  },
  toastIcon: {
    marginRight: 12,
  },
  toastMessage: {
    flex: 1,
    fontSize: 14,
    color: colors.gray900,
  }
})
```

#### Empty State
```tsx
<EmptyState
  icon="inbox"
  title="No messages yet"
  description="Start a conversation with your upline"
  action={{
    label: "Message Upline",
    onPress: handleMessageUpline
  }}
/>

// Styles
const styles = StyleSheet.create({
  emptyState: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: 32,
  },
  emptyIcon: {
    marginBottom: 16,
    opacity: 0.4,
  },
  emptyTitle: {
    fontSize: 18,
    fontWeight: '600',
    marginBottom: 8,
    textAlign: 'center',
  },
  emptyDescription: {
    fontSize: 14,
    color: colors.gray500,
    textAlign: 'center',
    marginBottom: 24,
  }
})
```

### 7. Loading States

#### Skeleton Loader
```tsx
<SkeletonLoader
  width="100%"
  height={100}
  borderRadius={12}
/>

// Styles
const styles = StyleSheet.create({
  skeleton: {
    backgroundColor: colors.gray100,
    overflow: 'hidden',
  },
  shimmer: {
    position: 'absolute',
    top: 0,
    left: 0,
    right: 0,
    bottom: 0,
    backgroundColor: colors.gray200,
    opacity: 0.5,
  }
})
```

## Layout Patterns

### Screen Layout
```tsx
const ScreenLayout = ({ children, title }) => (
  <SafeAreaView style={styles.container}>
    <View style={styles.header}>
      <Text style={styles.title}>{title}</Text>
    </View>
    <ScrollView style={styles.content}>
      {children}
    </ScrollView>
  </SafeAreaView>
)

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: colors.gray50,
  },
  header: {
    paddingHorizontal: 20,
    paddingVertical: 16,
    backgroundColor: colors.white,
    borderBottomWidth: 1,
    borderBottomColor: colors.gray100,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
  },
  content: {
    flex: 1,
    padding: 20,
  }
})
```

### Form Layout
```tsx
const FormLayout = ({ children, onSubmit }) => (
  <KeyboardAvoidingView behavior="padding">
    <ScrollView>
      <View style={styles.form}>
        {children}
      </View>
    </ScrollView>
  </KeyboardAvoidingView>
)

const styles = StyleSheet.create({
  form: {
    padding: 20,
  }
})
```

## Animation Guidelines

### Simple Transitions
```typescript
// Fade in animation
const fadeIn = {
  from: { opacity: 0 },
  to: { opacity: 1 },
  duration: 300,
}

// Scale animation for buttons
const buttonPress = {
  from: { scale: 1 },
  to: { scale: 0.95 },
  duration: 100,
}

// Slide in from bottom
const slideUp = {
  from: { translateY: 100 },
  to: { translateY: 0 },
  duration: 250,
}
```

## Accessibility

### Touch Targets
- Minimum size: 44x44 points
- Clear labels for screen readers
- Proper contrast ratios (4.5:1 minimum)
- Support for system font scaling

### Screen Reader Support
```tsx
<TouchableOpacity
  accessible
  accessibilityLabel="Submit content for grading"
  accessibilityHint="Double tap to submit your content"
  accessibilityRole="button"
>
  <Text>Submit</Text>
</TouchableOpacity>
```

## Dark Mode Support

### Automatic Theme Detection
```typescript
import { useColorScheme } from 'react-native'

const colors = {
  light: {
    background: '#FFFFFF',
    text: '#111827',
    border: '#D1D5DB',
  },
  dark: {
    background: '#1A1A2E',
    text: '#F3F4F6',
    border: '#373754',
  }
}

const useTheme = () => {
  const colorScheme = useColorScheme()
  return colors[colorScheme || 'light']
}
```

## Implementation Best Practices

### Component Structure
```typescript
// components/Button.tsx
import React from 'react'
import { TouchableOpacity, Text, ActivityIndicator } from 'react-native'
import { useTheme } from '../hooks/useTheme'

interface ButtonProps {
  title: string
  onPress: () => void
  variant?: 'primary' | 'secondary'
  loading?: boolean
  disabled?: boolean
}

export const Button: React.FC<ButtonProps> = ({
  title,
  onPress,
  variant = 'primary',
  loading = false,
  disabled = false,
}) => {
  const theme = useTheme()
  
  return (
    <TouchableOpacity
      style={[
        styles.button,
        variant === 'secondary' && styles.secondaryButton,
        disabled && styles.disabledButton,
      ]}
      onPress={onPress}
      disabled={disabled || loading}
    >
      {loading ? (
        <ActivityIndicator color={theme.white} />
      ) : (
        <Text style={styles.buttonText}>{title}</Text>
      )}
    </TouchableOpacity>
  )
}
```

This simplified design system focuses on the essential components needed for the Galaxy Team app, ensuring consistency while maintaining simplicity and ease of implementation.