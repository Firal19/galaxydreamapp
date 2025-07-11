# Galaxy Team Internal App - Complete Frontend Development Tickets

## FRONTEND-001: Project Setup & Routing Architecture

**Priority**: Critical  
**Estimated Hours**: 8  
**Dependencies**: None

### Objective
Setup Expo project with TypeScript, configure Expo Router for file-based routing, and establish the foundational architecture.

### Detailed Implementation Steps

#### 1. Initialize Expo Project
```bash
# Create new Expo app with TypeScript
npx create-expo-app@latest galaxy-team-app --template

# Navigate to project
cd galaxy-team-app

# Clear template files
rm -rf app && mkdir app
```

#### 2. Install Core Dependencies
```bash
# Navigation and routing
npx expo install expo-router expo-linking expo-constants expo-status-bar

# UI and styling
npx expo install react-native-safe-area-context react-native-screens 
npx expo install react-native-gesture-handler react-native-reanimated
npx expo install react-native-elements react-native-vector-icons
npx expo install expo-font @expo-google-fonts/inter

# Forms and validation
npm install react-hook-form zod @hookform/resolvers

# State management
npm install zustand immer

# Utilities
npx expo install expo-secure-store expo-clipboard @react-native-async-storage/async-storage
npx expo install expo-notifications expo-device expo-application

# Supabase
npm install @supabase/supabase-js

# Development
npm install -D @types/react @types/react-native
npm install -D prettier eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

#### 3. Configure TypeScript
```json
// tsconfig.json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["*"],
      "@/components/*": ["components/*"],
      "@/hooks/*": ["hooks/*"],
      "@/lib/*": ["lib/*"],
      "@/utils/*": ["utils/*"],
      "@/types/*": ["types/*"],
      "@/styles/*": ["styles/*"],
      "@/assets/*": ["assets/*"]
    }
  },
  "include": ["**/*.ts", "**/*.tsx", ".expo/types/**/*.ts", "expo-env.d.ts"]
}
```

#### 4. Setup Babel Configuration
```javascript
// babel.config.js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: [
      'expo-router/babel',
      'react-native-reanimated/plugin',
      [
        'module-resolver',
        {
          root: ['./'],
          alias: {
            '@': './',
            '@/components': './components',
            '@/hooks': './hooks',
            '@/lib': './lib',
            '@/utils': './utils',
            '@/types': './types',
            '@/styles': './styles',
            '@/assets': './assets'
          }
        }
      ]
    ]
  };
};
```

#### 5. Create App Configuration
```json
// app.json
{
  "expo": {
    "name": "Galaxy Team",
    "slug": "galaxy-team",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "automatic",
    "scheme": "galaxyteam",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#0B5CFF"
    },
    "assetBundlePatterns": ["**/*"],
    "ios": {
      "supportsTablet": false,
      "bundleIdentifier": "com.galaxyteam.app",
      "config": {
        "usesNonExemptEncryption": false
      }
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#0B5CFF"
      },
      "package": "com.galaxyteam.app",
      "permissions": ["NOTIFICATIONS", "READ_EXTERNAL_STORAGE", "WRITE_EXTERNAL_STORAGE"]
    },
    "web": {
      "favicon": "./assets/favicon.png",
      "bundler": "metro"
    },
    "plugins": [
      "expo-router",
      [
        "expo-notifications",
        {
          "icon": "./assets/notification-icon.png",
          "color": "#0B5CFF"
        }
      ]
    ]
  }
}
```

#### 6. Setup Routing Structure
```typescript
// app/_layout.tsx
import { useEffect } from 'react';
import { Stack } from 'expo-router';
import { ThemeProvider } from 'react-native-elements';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import { StatusBar } from 'expo-status-bar';
import * as SplashScreen from 'expo-splash-screen';
import { useFonts, Inter_400Regular, Inter_500Medium, Inter_600SemiBold, Inter_700Bold } from '@expo-google-fonts/inter';
import { useAuthStore } from '@/stores/authStore';
import { theme } from '@/styles/theme';

SplashScreen.preventAutoHideAsync();

export default function RootLayout() {
  const { initialize } = useAuthStore();
  
  const [fontsLoaded] = useFonts({
    Inter_400Regular,
    Inter_500Medium,
    Inter_600SemiBold,
    Inter_700Bold,
  });

  useEffect(() => {
    async function prepare() {
      await initialize();
      if (fontsLoaded) {
        await SplashScreen.hideAsync();
      }
    }
    prepare();
  }, [fontsLoaded]);

  if (!fontsLoaded) {
    return null;
  }

  return (
    <ThemeProvider theme={theme}>
      <SafeAreaProvider>
        <StatusBar style="auto" />
        <Stack>
          <Stack.Screen name="(auth)" options={{ headerShown: false }} />
          <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
          <Stack.Screen name="content/[id]" options={{ title: 'Content Details' }} />
          <Stack.Screen name="chat/[userId]" options={{ title: 'Chat' }} />
        </Stack>
      </SafeAreaProvider>
    </ThemeProvider>
  );
}

// app/index.tsx - Entry point with auth check
import { Redirect } from 'expo-router';
import { useAuthStore } from '@/stores/authStore';

export default function Index() {
  const { user, isLoading } = useAuthStore();

  if (isLoading) {
    return null; // Or loading screen
  }

  return <Redirect href={user ? '/(tabs)' : '/(auth)/login'} />;
}
```

#### 7. Create Theme Configuration
```typescript
// styles/theme.ts
export const colors = {
  primary: '#0B5CFF',
  secondary: '#00D4AA',
  accent: '#FFB800',
  success: '#10B981',
  error: '#EF4444',
  warning: '#F59E0B',
  info: '#3B82F6',
  
  // Grayscale
  black: '#000000',
  gray900: '#111827',
  gray700: '#374151',
  gray500: '#6B7280',
  gray300: '#D1D5DB',
  gray100: '#F3F4F6',
  white: '#FFFFFF',
  
  // Dark mode
  darkBg: '#1A1A2E',
  darkSurface: '#252542',
  darkBorder: '#373754',
};

export const spacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
  xxl: 48,
};

export const typography = {
  h1: {
    fontSize: 30,
    fontFamily: 'Inter_700Bold',
    lineHeight: 36,
  },
  h2: {
    fontSize: 24,
    fontFamily: 'Inter_600SemiBold',
    lineHeight: 30,
  },
  h3: {
    fontSize: 20,
    fontFamily: 'Inter_600SemiBold',
    lineHeight: 26,
  },
  body: {
    fontSize: 16,
    fontFamily: 'Inter_400Regular',
    lineHeight: 22,
  },
  caption: {
    fontSize: 14,
    fontFamily: 'Inter_400Regular',
    lineHeight: 18,
  },
  label: {
    fontSize: 12,
    fontFamily: 'Inter_500Medium',
    lineHeight: 16,
  },
};

export const theme = {
  colors,
  spacing,
  typography,
};
```

### Testing Requirements
- [ ] App launches without errors
- [ ] Fonts load correctly
- [ ] Navigation between screens works
- [ ] TypeScript compilation successful
- [ ] Path aliases resolve correctly

### Acceptance Criteria
- Project structure matches architecture document
- All dependencies installed and configured
- Routing system functional
- Theme system implemented
- No TypeScript errors

---

## FRONTEND-002: Authentication Screens Implementation

**Priority**: Critical  
**Estimated Hours**: 16  
**Dependencies**: FRONTEND-001

### Objective
Implement complete authentication flow including login, registration, OTP verification, and session management.

### Detailed Implementation Steps

#### 1. Create Auth Layout
```typescript
// app/(auth)/_layout.tsx
import { Stack } from 'expo-router';
import { View } from 'react-native';
import { colors } from '@/styles/theme';

export default function AuthLayout() {
  return (
    <View style={{ flex: 1, backgroundColor: colors.white }}>
      <Stack
        screenOptions={{
          headerStyle: {
            backgroundColor: colors.white,
          },
          headerTintColor: colors.gray900,
          headerTitleStyle: {
            fontFamily: 'Inter_600SemiBold',
          },
          headerShadowVisible: false,
        }}
      >
        <Stack.Screen 
          name="login" 
          options={{ 
            title: 'Welcome Back',
            headerShown: false 
          }} 
        />
        <Stack.Screen 
          name="register" 
          options={{ 
            title: 'Join Galaxy Team' 
          }} 
        />
        <Stack.Screen 
          name="verify-otp" 
          options={{ 
            title: 'Verify Phone Number' 
          }} 
        />
      </Stack>
    </View>
  );
}
```

#### 2. Login Screen Implementation
```typescript
// app/(auth)/login.tsx
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  KeyboardAvoidingView,
  Platform,
  ScrollView,
  Alert,
  ActivityIndicator,
} from 'react-native';
import { Link, router } from 'expo-router';
import { SafeAreaView } from 'react-native-safe-area-context';
import { Ionicons } from '@expo/vector-icons';
import { useForm, Controller } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';
import { supabase } from '@/lib/supabase';
import { colors, spacing, typography } from '@/styles/theme';
import { formatPhoneNumber } from '@/utils/formatters';

const loginSchema = z.object({
  phone: z.string()
    .min(9, 'Phone number must be at least 9 digits')
    .regex(/^[0-9]+$/, 'Phone number must contain only digits'),
});

type LoginForm = z.infer<typeof loginSchema>;

export default function LoginScreen() {
  const [isLoading, setIsLoading] = useState(false);
  
  const { control, handleSubmit, formState: { errors } } = useForm<LoginForm>({
    resolver: zodResolver(loginSchema),
    defaultValues: {
      phone: '',
    },
  });

  const onSubmit = async (data: LoginForm) => {
    setIsLoading(true);
    try {
      const formattedPhone = formatPhoneNumber(data.phone);
      
      // Send OTP
      const { error } = await supabase.auth.signInWithOtp({
        phone: formattedPhone,
      });

      if (error) throw error;

      // Navigate to OTP verification with phone
      router.push({
        pathname: '/(auth)/verify-otp',
        params: { phone: formattedPhone },
      });
    } catch (error: any) {
      Alert.alert('Error', error.message || 'Failed to send OTP');
    } finally {
      setIsLoading(false);
    }
  };

  return (
    <SafeAreaView style={styles.container}>
      <KeyboardAvoidingView
        behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
        style={{ flex: 1 }}
      >
        <ScrollView 
          contentContainerStyle={styles.scrollContent}
          showsVerticalScrollIndicator={false}
        >
          {/* Logo Section */}
          <View style={styles.logoSection}>
            <View style={styles.logoContainer}>
              <Ionicons name="rocket" size={60} color={colors.primary} />
            </View>
            <Text style={styles.title}>Galaxy Team</Text>
            <Text style={styles.subtitle}>Build Your Success Network</Text>
          </View>

          {/* Form Section */}
          <View style={styles.formSection}>
            <Text style={styles.formTitle}>Welcome Back</Text>
            <Text style={styles.formSubtitle}>
              Enter your phone number to continue
            </Text>

            <View style={styles.inputContainer}>
              <Text style={styles.label}>Phone Number</Text>
              <View style={styles.phoneInputWrapper}>
                <View style={styles.countryCode}>
                  <Text style={styles.countryCodeText}>+251</Text>
                </View>
                <Controller
                  control={control}
                  name="phone"
                  render={({ field: { onChange, onBlur, value } }) => (
                    <TextInput
                      style={[
                        styles.phoneInput,
                        errors.phone && styles.inputError
                      ]}
                      placeholder="912345678"
                      placeholderTextColor={colors.gray500}
                      keyboardType="phone-pad"
                      maxLength={9}
                      onBlur={onBlur}
                      onChangeText={onChange}
                      value={value}
                    />
                  )}
                />
              </View>
              {errors.phone && (
                <Text style={styles.errorText}>{errors.phone.message}</Text>
              )}
            </View>

            <TouchableOpacity
              style={[styles.button, isLoading && styles.buttonDisabled]}
              onPress={handleSubmit(onSubmit)}
              disabled={isLoading}
            >
              {isLoading ? (
                <ActivityIndicator color={colors.white} />
              ) : (
                <Text style={styles.buttonText}>Send Verification Code</Text>
              )}
            </TouchableOpacity>

            <View style={styles.linkContainer}>
              <Text style={styles.linkText}>Don't have an account? </Text>
              <Link href="/(auth)/register" asChild>
                <TouchableOpacity>
                  <Text style={styles.link}>Join Galaxy Team</Text>
                </TouchableOpacity>
              </Link>
            </View>
          </View>
        </ScrollView>
      </KeyboardAvoidingView>
    </SafeAreaView>
  );
}

const styles = {
  container: {
    flex: 1,
    backgroundColor: colors.white,
  },
  scrollContent: {
    flexGrow: 1,
    paddingHorizontal: spacing.lg,
  },
  logoSection: {
    alignItems: 'center',
    paddingTop: spacing.xxl,
    paddingBottom: spacing.xl,
  },
  logoContainer: {
    width: 120,
    height: 120,
    borderRadius: 60,
    backgroundColor: colors.primary + '10',
    alignItems: 'center',
    justifyContent: 'center',
    marginBottom: spacing.md,
  },
  title: {
    ...typography.h1,
    color: colors.gray900,
    marginBottom: spacing.xs,
  },
  subtitle: {
    ...typography.body,
    color: colors.gray500,
  },
  formSection: {
    flex: 1,
  },
  formTitle: {
    ...typography.h2,
    color: colors.gray900,
    marginBottom: spacing.xs,
  },
  formSubtitle: {
    ...typography.body,
    color: colors.gray500,
    marginBottom: spacing.xl,
  },
  inputContainer: {
    marginBottom: spacing.lg,
  },
  label: {
    ...typography.label,
    color: colors.gray700,
    marginBottom: spacing.xs,
    textTransform: 'uppercase',
  },
  phoneInputWrapper: {
    flexDirection: 'row',
    borderWidth: 1,
    borderColor: colors.gray300,
    borderRadius: 8,
    overflow: 'hidden',
  },
  countryCode: {
    backgroundColor: colors.gray100,
    paddingHorizontal: spacing.md,
    justifyContent: 'center',
    borderRightWidth: 1,
    borderRightColor: colors.gray300,
  },
  countryCodeText: {
    ...typography.body,
    color: colors.gray700,
    fontFamily: 'Inter_500Medium',
  },
  phoneInput: {
    flex: 1,
    paddingHorizontal: spacing.md,
    paddingVertical: spacing.md,
    ...typography.body,
    color: colors.gray900,
  },
  inputError: {
    borderColor: colors.error,
  },
  errorText: {
    ...typography.caption,
    color: colors.error,
    marginTop: spacing.xs,
  },
  button: {
    backgroundColor: colors.primary,
    paddingVertical: spacing.md,
    borderRadius: 8,
    alignItems: 'center',
    marginBottom: spacing.lg,
  },
  buttonDisabled: {
    opacity: 0.7,
  },
  buttonText: {
    ...typography.body,
    color: colors.white,
    fontFamily: 'Inter_600SemiBold',
  },
  linkContainer: {
    flexDirection: 'row',
    justifyContent: 'center',
    alignItems: 'center',
  },
  linkText: {
    ...typography.body,
    color: colors.gray500,
  },
  link: {
    ...typography.body,
    color: colors.primary,
    fontFamily: 'Inter_600SemiBold',
  },
};
```

#### 3. Registration Screen
```typescript
// app/(auth)/register.tsx
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  ScrollView,
  Alert,
  ActivityIndicator,
} from 'react-native';
import { router } from 'expo-router';
import { SafeAreaView } from 'react-native-safe-area-context';
import { Ionicons } from '@expo/vector-icons';
import { useForm, Controller } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';
import { supabase } from '@/lib/supabase';
import { colors, spacing, typography } from '@/styles/theme';
import { formatPhoneNumber } from '@/utils/formatters';
import { Picker } from '@react-native-picker/picker';

const registerSchema = z.object({
  fullName: z.string().min(3, 'Full name must be at least 3 characters'),
  phone: z.string()
    .min(9, 'Phone number must be at least 9 digits')
    .regex(/^[0-9]+$/, 'Phone number must contain only digits'),
  officeId: z.string().min(1, 'Please select an office'),
  uplinePhone: z.string().optional(),
  agreeToTerms: z.boolean().refine(val => val === true, {
    message: 'You must agree to the terms and conditions',
  }),
});

type RegisterForm = z.infer<typeof registerSchema>;

interface Office {
  id: string;
  name: string;
  city: string;
}

export default function RegisterScreen() {
  const [isLoading, setIsLoading] = useState(false);
  const [offices, setOffices] = useState<Office[]>([]);
  const [loadingOffices, setLoadingOffices] = useState(true);

  const { control, handleSubmit, formState: { errors }, watch } = useForm<RegisterForm>({
    resolver: zodResolver(registerSchema),
    defaultValues: {
      fullName: '',
      phone: '',
      officeId: '',
      uplinePhone: '',
      agreeToTerms: false,
    },
  });

  useEffect(() => {
    fetchOffices();
  }, []);

  const fetchOffices = async () => {
    try {
      const { data, error } = await supabase
        .from('offices')
        .select('id, name, city')
        .order('name');

      if (error) throw error;
      setOffices(data || []);
    } catch (error) {
      Alert.alert('Error', 'Failed to load offices');
    } finally {
      setLoadingOffices(false);
    }
  };

  const onSubmit = async (data: RegisterForm) => {
    setIsLoading(true);
    try {
      const formattedPhone = formatPhoneNumber(data.phone);
      
      // Check if phone number already exists
      const { data: existingUser } = await supabase
        .from('profiles')
        .select('id')
        .eq('phone_number', formattedPhone)
        .single();

      if (existingUser) {
        Alert.alert('Error', 'This phone number is already registered');
        return;
      }

      // Store registration data temporarily
      await AsyncStorage.setItem('pendingRegistration', JSON.stringify({
        fullName: data.fullName,
        phone: formattedPhone,
        officeId: data.officeId,
        uplinePhone: data.uplinePhone ? formatPhoneNumber(data.uplinePhone) : null,
      }));

      // Send OTP
      const { error } = await supabase.auth.signInWithOtp({
        phone: formattedPhone,
      });

      if (error) throw error;

      // Navigate to OTP verification
      router.push({
        pathname: '/(auth)/verify-otp',
        params: { 
          phone: formattedPhone,
          isRegistration: 'true'
        },
      });
    } catch (error: any) {
      Alert.alert('Error', error.message || 'Failed to register');
    } finally {
      setIsLoading(false);
    }
  };

  const agreeToTerms = watch('agreeToTerms');

  return (
    <SafeAreaView style={styles.container}>
      <ScrollView 
        showsVerticalScrollIndicator={false}
        contentContainerStyle={styles.scrollContent}
      >
        <View style={styles.header}>
          <Text style={styles.title}>Join Galaxy Team</Text>
          <Text style={styles.subtitle}>
            Start your journey to financial freedom
          </Text>
        </View>

        <View style={styles.form}>
          {/* Full Name Input */}
          <View style={styles.inputContainer}>
            <Text style={styles.label}>Full Name</Text>
            <Controller
              control={control}
              name="fullName"
              render={({ field: { onChange, onBlur, value } }) => (
                <TextInput
                  style={[styles.input, errors.fullName && styles.inputError]}
                  placeholder="Enter your full name"
                  placeholderTextColor={colors.gray500}
                  onBlur={onBlur}
                  onChangeText={onChange}
                  value={value}
                  autoCapitalize="words"
                />
              )}
            />
            {errors.fullName && (
              <Text style={styles.errorText}>{errors.fullName.message}</Text>
            )}
          </View>

          {/* Phone Number Input */}
          <View style={styles.inputContainer}>
            <Text style={styles.label}>Phone Number</Text>
            <View style={styles.phoneInputWrapper}>
              <View style={styles.countryCode}>
                <Text style={styles.countryCodeText}>+251</Text>
              </View>
              <Controller
                control={control}
                name="phone"
                render={({ field: { onChange, onBlur, value } }) => (
                  <TextInput
                    style={[styles.phoneInput, errors.phone && styles.inputError]}
                    placeholder="912345678"
                    placeholderTextColor={colors.gray500}
                    keyboardType="phone-pad"
                    maxLength={9}
                    onBlur={onBlur}
                    onChangeText={onChange}
                    value={value}
                  />
                )}
              />
            </View>
            {errors.phone && (
              <Text style={styles.errorText}>{errors.phone.message}</Text>
            )}
          </View>

          {/* Office Selection */}
          <View style={styles.inputContainer}>
            <Text style={styles.label}>Select Office</Text>
            {loadingOffices ? (
              <View style={styles.loadingContainer}>
                <ActivityIndicator size="small" color={colors.primary} />
              </View>
            ) : (
              <Controller
                control={control}
                name="officeId"
                render={({ field: { onChange, value } }) => (
                  <View style={[styles.pickerContainer, errors.officeId && styles.inputError]}>
                    <Picker
                      selectedValue={value}
                      onValueChange={onChange}
                      style={styles.picker}
                    >
                      <Picker.Item label="Choose your nearest office" value="" />
                      {offices.map((office) => (
                        <Picker.Item
                          key={office.id}
                          label={`${office.name} - ${office.city}`}
                          value={office.id}
                        />
                      ))}
                    </Picker>
                  </View>
                )}
              />
            )}
            {errors.officeId && (
              <Text style={styles.errorText}>{errors.officeId.message}</Text>
            )}
          </View>

          {/* Upline Phone (Optional) */}
          <View style={styles.inputContainer}>
            <Text style={styles.label}>Upline Phone Number (Optional)</Text>
            <View style={styles.phoneInputWrapper}>
              <View style={styles.countryCode}>
                <Text style={styles.countryCodeText}>+251</Text>
              </View>
              <Controller
                control={control}
                name="uplinePhone"
                render={({ field: { onChange, onBlur, value } }) => (
                  <TextInput
                    style={styles.phoneInput}
                    placeholder="Who invited you?"
                    placeholderTextColor={colors.gray500}
                    keyboardType="phone-pad"
                    maxLength={9}
                    onBlur={onBlur}
                    onChangeText={onChange}
                    value={value}
                  />
                )}
              />
            </View>
          </View>

          {/* Terms Agreement */}
          <Controller
            control={control}
            name="agreeToTerms"
            render={({ field: { onChange, value } }) => (
              <TouchableOpacity
                style={styles.checkboxContainer}
                onPress={() => onChange(!value)}
              >
                <View style={[styles.checkbox, value && styles.checkboxChecked]}>
                  {value && <Ionicons name="checkmark" size={16} color={colors.white} />}
                </View>
                <Text style={styles.checkboxLabel}>
                  I agree to the Terms of Service and Privacy Policy
                </Text>
              </TouchableOpacity>
            )}
          />
          {errors.agreeToTerms && (
            <Text style={styles.errorText}>{errors.agreeToTerms.message}</Text>
          )}

          {/* Submit Button */}
          <TouchableOpacity
            style={[
              styles.button,
              (!agreeToTerms || isLoading) && styles.buttonDisabled
            ]}
            onPress={handleSubmit(onSubmit)}
            disabled={!agreeToTerms || isLoading}
          >
            {isLoading ? (
              <ActivityIndicator color={colors.white} />
            ) : (
              <Text style={styles.buttonText}>Create Account</Text>
            )}
          </TouchableOpacity>
        </View>
      </ScrollView>
    </SafeAreaView>
  );
}

// Similar styles structure as login screen
```

#### 4. OTP Verification Screen
```typescript
// app/(auth)/verify-otp.tsx
import React, { useState, useRef, useEffect } from 'react';
import {
  View,
  Text,
  TextInput,
  TouchableOpacity,
  Alert,
  ActivityIndicator,
} from 'react-native';
import { router, useLocalSearchParams } from 'expo-router';
import { SafeAreaView } from 'react-native-safe-area-context';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { supabase } from '@/lib/supabase';
import { useAuthStore } from '@/stores/authStore';
import { colors, spacing, typography } from '@/styles/theme';

export default function VerifyOTPScreen() {
  const { phone, isRegistration } = useLocalSearchParams<{
    phone: string;
    isRegistration?: string;
  }>();
  
  const [otp, setOtp] = useState(['', '', '', '', '', '']);
  const [isLoading, setIsLoading] = useState(false);
  const [countdown, setCountdown] = useState(60);
  const [canResend, setCanResend] = useState(false);
  
  const inputRefs = useRef<TextInput[]>([]);
  const { setUser } = useAuthStore();

  useEffect(() => {
    const timer = setInterval(() => {
      setCountdown((prev) => {
        if (prev <= 1) {
          setCanResend(true);
          return 0;
        }
        return prev - 1;
      });
    }, 1000);

    return () => clearInterval(timer);
  }, []);

  const handleOtpChange = (value: string, index: number) => {
    const newOtp = [...otp];
    newOtp[index] = value;
    setOtp(newOtp);

    // Auto-focus next input
    if (value && index < 5) {
      inputRefs.current[index + 1]?.focus();
    }

    // Auto-submit when all digits entered
    if (newOtp.every(digit => digit) && index === 5) {
      verifyOTP(newOtp.join(''));
    }
  };

  const handleKeyPress = (key: string, index: number) => {
    if (key === 'Backspace' && !otp[index] && index > 0) {
      inputRefs.current[index - 1]?.focus();
    }
  };

  const verifyOTP = async (otpCode: string) => {
    setIsLoading(true);
    try {
      const { data, error } = await supabase.auth.verifyOtp({
        phone: phone,
        token: otpCode,
        type: 'sms',
      });

      if (error) throw error;

      if (isRegistration === 'true' && data.user) {
        // Complete registration
        const pendingData = await AsyncStorage.getItem('pendingRegistration');
        if (pendingData) {
          const registrationData = JSON.parse(pendingData);
          
          // Create profile
          const { error: profileError } = await supabase
            .from('profiles')
            .insert({
              id: data.user.id,
              full_name: registrationData.fullName,
              phone_number: registrationData.phone,
              office_id: registrationData.officeId,
              upline_phone: registrationData.uplinePhone,
            });

          if (profileError) throw profileError;

          // Clean up
          await AsyncStorage.removeItem('pendingRegistration');
        }
      }

      // Fetch user profile
      const { data: profile } = await supabase
        .from('profiles')
        .select('*, office:offices(*)')
        .eq('id', data.user!.id)
        .single();

      if (profile) {
        setUser(profile);
        router.replace('/(tabs)');
      }
    } catch (error: any) {
      Alert.alert('Error', error.message || 'Invalid verification code');
      setOtp(['', '', '', '', '', '']);
      inputRefs.current[0]?.focus();
    } finally {
      setIsLoading(false);
    }
  };

  const resendOTP = async () => {
    try {
      const { error } = await supabase.auth.signInWithOtp({
        phone: phone,
      });

      if (error) throw error;

      Alert.alert('Success', 'Verification code sent again');
      setCountdown(60);
      setCanResend(false);
    } catch (error: any) {
      Alert.alert('Error', error.message || 'Failed to resend code');
    }
  };

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.content}>
        <View style={styles.header}>
          <Text style={styles.title}>Verify Phone Number</Text>
          <Text style={styles.subtitle}>
            Enter the 6-digit code sent to
          </Text>
          <Text style={styles.phone}>{phone}</Text>
        </View>

        <View style={styles.otpContainer}>
          {otp.map((digit, index) => (
            <TextInput
              key={index}
              ref={(ref) => ref && (inputRefs.current[index] = ref)}
              style={[
                styles.otpInput,
                digit && styles.otpInputFilled,
              ]}
              value={digit}
              onChangeText={(value) => handleOtpChange(value, index)}
              onKeyPress={({ nativeEvent }) => handleKeyPress(nativeEvent.key, index)}
              keyboardType="number-pad"
              maxLength={1}
              selectTextOnFocus
            />
          ))}
        </View>

        {!canResend && (
          <Text style={styles.countdown}>
            Resend code in {countdown}s
          </Text>
        )}

        {canResend && (
          <TouchableOpacity onPress={resendOTP}>
            <Text style={styles.resendLink}>Resend verification code</Text>
          </TouchableOpacity>
        )}

        {isLoading && (
          <View style={styles.loadingContainer}>
            <ActivityIndicator size="large" color={colors.primary} />
            <Text style={styles.loadingText}>Verifying...</Text>
          </View>
        )}
      </View>
    </SafeAreaView>
  );
}

const styles = {
  // OTP specific styles
  otpContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginVertical: spacing.xl,
  },
  otpInput: {
    width: 50,
    height: 60,
    borderWidth: 2,
    borderColor: colors.gray300,
    borderRadius: 8,
    textAlign: 'center',
    fontSize: 24,
    fontFamily: 'Inter_600SemiBold',
    color: colors.gray900,
  },
  otpInputFilled: {
    borderColor: colors.primary,
    backgroundColor: colors.primary + '10',
  },
  // ... other styles
};
```

### Testing Requirements
- [ ] Phone number validation works
- [ ] OTP is sent successfully
- [ ] Registration creates profile correctly
- [ ] Session persists after login
- [ ] Error handling displays properly

---

## FRONTEND-003: Tab Navigation & Home Dashboard

**Priority**: High  
**Estimated Hours**: 12  
**Dependencies**: FRONTEND-002

### Objective
Implement the main tab navigation and create a comprehensive home dashboard showing user stats and quick actions.

### Detailed Implementation Steps

#### 1. Tab Navigator Layout
```typescript
// app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router';
import { Ionicons } from '@expo/vector-icons';
import { View, Text, Platform } from 'react-native';
import { colors, typography } from '@/styles/theme';

export default function TabLayout() {
  return (
    <Tabs
      screenOptions={{
        tabBarActiveTintColor: colors.primary,
        tabBarInactiveTintColor: colors.gray500,
        tabBarStyle: {
          backgroundColor: colors.white,
          borderTopWidth: 1,
          borderTopColor: colors.gray100,
          paddingBottom: Platform.OS === 'ios' ? 20 : 5,
          paddingTop: 5,
          height: Platform.OS === 'ios' ? 85 : 65,
        },
        tabBarLabelStyle: {
          ...typography.label,
          fontSize: 11,
        },
        headerStyle: {
          backgroundColor: colors.white,
        },
        headerTintColor: colors.gray900,
        headerTitleStyle: {
          ...typography.h3,
        },
        headerShadowVisible: false,
      }}
    >
      <Tabs.Screen
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="home" size={size} color={color} />
          ),
          headerShown: false,
        }}
      />
      <Tabs.Screen
        name="content"
        options={{
          title: 'Content',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="document-text" size={size} color={color} />
          ),
        }}
      />
      <Tabs.Screen
        name="grade"
        options={{
          title: 'Grade',
          tabBarIcon: ({ color, size }) => (
            <View style={{ position: 'relative' }}>
              <Ionicons name="checkmark-circle" size={size} color={color} />
              <View style={styles.centralTabIndicator} />
            </View>
          ),
        }}
      />
      <Tabs.Screen
        name="analytics"
        options={{
          title: 'Analytics',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="bar-chart" size={size} color={color} />
          ),
        }}
      />
      <Tabs.Screen
        name="messages"
        options={{
          title: 'Messages',
          tabBarIcon: ({ color, size }) => (
            <Ionicons name="chatbubbles" size={size} color={color} />
          ),
          tabBarBadge: undefined, // Will be set dynamically
        }}
      />
    </Tabs>
  );
}

const styles = {
  centralTabIndicator: {
    position: 'absolute',
    top: -2,
    right: -2,
    width: 8,
    height: 8,
    borderRadius: 4,
    backgroundColor: colors.secondary,
  },
};
```

#### 2. Home Dashboard Implementation
```typescript
// app/(tabs)/index.tsx
import React, { useEffect, useState } from 'react';
import {
  View,
  Text,
  ScrollView,
  TouchableOpacity,
  RefreshControl,
  ActivityIndicator,
} from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { router } from 'expo-router';
import { Ionicons } from '@expo/vector-icons';
import { LinearGradient } from 'expo-linear-gradient';
import { useAuthStore } from '@/stores/authStore';
import { supabase } from '@/lib/supabase';
import { colors, spacing, typography } from '@/styles/theme';
import { formatNumber } from '@/utils/formatters';

interface DashboardStats {
  todayClicks: number;
  weekClicks: number;
  totalPosts: number;
  passRate: number;
  pendingContent: number;
  unreadMessages: number;
}

export default function HomeScreen() {
  const { user } = useAuthStore();
  const [stats, setStats] = useState<DashboardStats>({
    todayClicks: 0,
    weekClicks: 0,
    totalPosts: 0,
    passRate: 0,
    pendingContent: 0,
    unreadMessages: 0,
  });
  const [loading, setLoading] = useState(true);
  const [refreshing, setRefreshing] = useState(false);

  useEffect(() => {
    fetchDashboardData();
    setupRealtimeSubscriptions();
  }, []);

  const fetchDashboardData = async () => {
    try {
      // Fetch today's clicks
      const today = new Date().toISOString().split('T')[0];
      const { data: todayAnalytics } = await supabase
        .from('link_analytics')
        .select('total_clicks')
        .eq('user_id', user?.id)
        .gte('last_clicked', today);

      const todayClicks = todayAnalytics?.reduce(
        (sum, item) => sum + item.total_clicks,
        0
      ) || 0;

      // Fetch week's data
      const weekAgo = new Date();
      weekAgo.setDate(weekAgo.getDate() - 7);
      const { data: weekAnalytics } = await supabase
        .from('link_analytics')
        .select('total_clicks')
        .eq('user_id', user?.id)
        .gte('last_clicked', weekAgo.toISOString());

      const weekClicks = weekAnalytics?.reduce(
        (sum, item) => sum + item.total_clicks,
        0
      ) || 0;

      // Fetch content stats
      const { data: contentStats, count: totalPosts } = await supabase
        .from('content_submissions')
        .select('passed', { count: 'exact' })
        .eq('user_id', user?.id);

      const passedPosts = contentStats?.filter(c => c.passed).length || 0;
      const passRate = totalPosts ? (passedPosts / totalPosts) * 100 : 0;

      // Fetch pending content ideas
      const { count: pendingContent } = await supabase
        .from('content_ideas')
        .select('*', { count: 'exact', head: true })
        .eq('active', true)
        .not('id', 'in', `(
          SELECT content_idea_id 
          FROM content_submissions 
          WHERE user_id = '${user?.id}'
        )`);

      // Fetch unread messages
      const { count: unreadMessages } = await supabase
        .from('messages')
        .select('*', { count: 'exact', head: true })
        .eq('recipient_id', user?.id)
        .eq('read', false);

      setStats({
        todayClicks,
        weekClicks,
        totalPosts: totalPosts || 0,
        passRate,
        pendingContent: pendingContent || 0,
        unreadMessages: unreadMessages || 0,
      });
    } catch (error) {
      console.error('Error fetching dashboard data:', error);
    } finally {
      setLoading(false);
      setRefreshing(false);
    }
  };

  const setupRealtimeSubscriptions = () => {
    // Subscribe to link clicks
    const analyticsSubscription = supabase
      .channel('dashboard_analytics')
      .on(
        'postgres_changes',
        {
          event: 'UPDATE',
          schema: 'public',
          table: 'link_analytics',
          filter: `user_id=eq.${user?.id}`,
        },
        () => {
          fetchDashboardData();
        }
      )
      .subscribe();

    // Subscribe to new messages
    const messagesSubscription = supabase
      .channel('dashboard_messages')
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          table: 'messages',
          filter: `recipient_id=eq.${user?.id}`,
        },
        () => {
          setStats(prev => ({
            ...prev,
            unreadMessages: prev.unreadMessages + 1,
          }));
        }
      )
      .subscribe();

    return () => {
      analyticsSubscription.unsubscribe();
      messagesSubscription.unsubscribe();
    };
  };

  const onRefresh = () => {
    setRefreshing(true);
    fetchDashboardData();
  };

  if (loading) {
    return (
      <SafeAreaView style={styles.container}>
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="large" color={colors.primary} />
        </View>
      </SafeAreaView>
    );
  }

  return (
    <SafeAreaView style={styles.container}>
      <ScrollView
        showsVerticalScrollIndicator={false}
        refreshControl={
          <RefreshControl
            refreshing={refreshing}
            onRefresh={onRefresh}
            tintColor={colors.primary}
          />
        }
      >
        {/* Header Section */}
        <LinearGradient
          colors={[colors.primary, colors.primary + 'DD']}
          style={styles.header}
        >
          <View style={styles.headerContent}>
            <View>
              <Text style={styles.greeting}>
                Welcome back,
              </Text>
              <Text style={styles.userName}>
                {user?.full_name}
              </Text>
              <Text style={styles.userRole}>
                {user?.role === 'leader' ? 'Team Leader' : 'Galaxy Member'}
              </Text>
            </View>
            <TouchableOpacity
              style={styles.profileButton}
              onPress={() => router.push('/profile')}
            >
              <Ionicons name="person-circle" size={50} color={colors.white} />
            </TouchableOpacity>
          </View>
        </LinearGradient>

        {/* Quick Stats */}
        <View style={styles.statsGrid}>
          <TouchableOpacity
            style={styles.statCard}
            onPress={() => router.push('/(tabs)/analytics')}
          >
            <View style={[styles.statIcon, { backgroundColor: colors.secondary + '20' }]}>
              <Ionicons name="trending-up" size={24} color={colors.secondary} />
            </View>
            <Text style={styles.statValue}>{formatNumber(stats.todayClicks)}</Text>
            <Text style={styles.statLabel}>Today's Clicks</Text>
          </TouchableOpacity>

          <TouchableOpacity
            style={styles.statCard}
            onPress={() => router.push('/(tabs)/analytics')}
          >
            <View style={[styles.statIcon, { backgroundColor: colors.primary + '20' }]}>
              <Ionicons name="calendar" size={24} color={colors.primary} />
            </View>
            <Text style={styles.statValue}>{formatNumber(stats.weekClicks)}</Text>
            <Text style={styles.statLabel}>Week Clicks</Text>
          </TouchableOpacity>

          <TouchableOpacity
            style={styles.statCard}
            onPress={() => router.push('/(tabs)/content')}
          >
            <View style={[styles.statIcon, { backgroundColor: colors.accent + '20' }]}>
              <Ionicons name="document-text" size={24} color={colors.accent} />
            </View>
            <Text style={styles.statValue}>{stats.totalPosts}</Text>
            <Text style={styles.statLabel}>Total Posts</Text>
          </TouchableOpacity>

          <View style={styles.statCard}>
            <View style={[styles.statIcon, { backgroundColor: colors.success + '20' }]}>
              <Ionicons name="checkmark-circle" size={24} color={colors.success} />
            </View>
            <Text style={styles.statValue}>{stats.passRate.toFixed(0)}%</Text>
            <Text style={styles.statLabel}>Pass Rate</Text>
          </View>
        </View>

        {/* Quick Actions */}
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Quick Actions</Text>
          
          {stats.pendingContent > 0 && (
            <TouchableOpacity
              style={styles.actionCard}
              onPress={() => router.push('/(tabs)/content')}
            >
              <View style={styles.actionIcon}>
                <Ionicons name="create" size={24} color={colors.primary} />
              </View>
              <View style={styles.actionContent}>
                <Text style={styles.actionTitle}>
                  New Content Ideas Available
                </Text>
                <Text style={styles.actionSubtitle}>
                  {stats.pendingContent} ideas waiting for you
                </Text>
              </View>
              <Ionicons name="chevron-forward" size={20} color={colors.gray500} />
            </TouchableOpacity>
          )}

          <TouchableOpacity
            style={styles.actionCard}
            onPress={() => router.push('/(tabs)/grade')}
          >
            <View style={styles.actionIcon}>
              <Ionicons name="checkmark-done" size={24} color={colors.secondary} />
            </View>
            <View style={styles.actionContent}>
              <Text style={styles.actionTitle}>
                Grade Your Content
              </Text>
              <Text style={styles.actionSubtitle}>
                Check quality before posting
              </Text>
            </View>
            <Ionicons name="chevron-forward" size={20} color={colors.gray500} />
          </TouchableOpacity>

          {stats.unreadMessages > 0 && (
            <TouchableOpacity
              style={[styles.actionCard, styles.actionCardHighlight]}
              onPress={() => router.push('/(tabs)/messages')}
            >
              <View style={styles.actionIcon}>
                <Ionicons name="mail" size={24} color={colors.error} />
                <View style={styles.badge}>
                  <Text style={styles.badgeText}>{stats.unreadMessages}</Text>
                </View>
              </View>
              <View style={styles.actionContent}>
                <Text style={styles.actionTitle}>
                  Unread Messages
                </Text>
                <Text style={styles.actionSubtitle}>
                  Check your messages now
                </Text>
              </View>
              <Ionicons name="chevron-forward" size={20} color={colors.gray500} />
            </TouchableOpacity>
          )}
        </View>

        {/* Recent Activity */}
        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Recent Activity</Text>
          <RecentActivityList userId={user?.id} />
        </View>
      </ScrollView>
    </SafeAreaView>
  );
}

// Recent Activity Component
const RecentActivityList = ({ userId }: { userId?: string }) => {
  const [activities, setActivities] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetchRecentActivities();
  }, []);

  const fetchRecentActivities = async () => {
    try {
      const { data } = await supabase
        .from('content_submissions')
        .select(`
          id,
          created_at,
          passed,
          tracking_id,
          link_analytics(total_clicks)
        `)
        .eq('user_id', userId)
        .order('created_at', { ascending: false })
        .limit(5);

      setActivities(data || []);
    } catch (error) {
      console.error('Error fetching activities:', error);
    } finally {
      setLoading(false);
    }
  };

  if (loading) {
    return <ActivityIndicator size="small" color={colors.primary} />;
  }

  return (
    <View>
      {activities.map((activity: any) => (
        <View key={activity.id} style={styles.activityItem}>
          <View style={styles.activityIcon}>
            <Ionicons
              name={activity.passed ? 'checkmark-circle' : 'close-circle'}
              size={20}
              color={activity.passed ? colors.success : colors.error}
            />
          </View>
          <View style={styles.activityContent}>
            <Text style={styles.activityTitle}>
              Content {activity.passed ? 'Passed' : 'Failed'} Grading
            </Text>
            <Text style={styles.activityTime}>
              {new Date(activity.created_at).toLocaleDateString()}
            </Text>
          </View>
          {activity.link_analytics?.total_clicks > 0 && (
            <Text style={styles.activityClicks}>
              {activity.link_analytics.total_clicks} clicks
            </Text>
          )}
        </View>
      ))}
    </View>
  );
};

// Styles object would be extensive - includes all component styles
```

### Testing Requirements
- [ ] Tab navigation works smoothly
- [ ] Dashboard stats are accurate
- [ ] Real-time updates function
- [ ] Pull-to-refresh works
- [ ] Quick actions navigate correctly

---

## FRONTEND-004: Content Hub & Distribution

**Priority**: High  
**Estimated Hours**: 12  
**Dependencies**: FRONTEND-003

### Objective
Build the content hub where members view content ideas from leaders and track their content creation progress.

### Implementation Details
[Complete implementation with ContentHub screen, ContentCard components, real-time updates, etc.]

---

## FRONTEND-005: Content Grading System

**Priority**: High  
**Estimated Hours**: 10  
**Dependencies**: FRONTEND-004

### Objective
Implement the 4-point content grading system with real-time validation and tracking link generation.

### Implementation Details
[Complete grading screen with validation logic, ChecklistItem components, tracking ID generation, etc.]

---

## FRONTEND-006: Analytics Dashboard

**Priority**: High  
**Estimated Hours**: 10  
**Dependencies**: FRONTEND-005

### Objective
Create simple analytics display showing click counts, best performing posts, and lead notifications.

### Implementation Details
[Analytics screen with charts, metrics display, real-time updates, etc.]

---

## FRONTEND-007: Messaging & Communication

**Priority**: Medium  
**Estimated Hours**: 12  
**Dependencies**: FRONTEND-003

### Objective
Implement messaging system for upline communication and viewing announcements.

### Implementation Details
[Messages list, chat screen, real-time messaging, push notifications, etc.]

---

## FRONTEND-008: Component Library & Design System

**Priority**: Medium  
**Estimated Hours**: 8  
**Dependencies**: FRONTEND-001

### Objective
Create reusable UI components following the design system.

### Implementation Details
[All UI components: Button, Card, Input, MetricCard, Toast, EmptyState, etc.]

---

## FRONTEND-009: State Management & Data Synchronization

**Priority**: High  
**Estimated Hours**: 10  
**Dependencies**: FRONTEND-008

### Objective
Implement Zustand stores and Supabase real-time synchronization.

### Implementation Details
[Auth store, content store, analytics store, real-time subscriptions, offline support, etc.]

---

## FRONTEND-010: Performance Optimization & Polish

**Priority**: High  
**Estimated Hours**: 12  
**Dependencies**: All previous tickets

### Objective
Optimize app performance, implement error handling, and add final polish.

### Implementation Details
[Memoization, lazy loading, error boundaries, splash screen, app icons, testing, etc.]

---

## FRONTEND-004: Content Hub & Distribution

**Priority**: High  
**Estimated Hours**: 12  
**Dependencies**: FRONTEND-003

### Objective
Build the content hub where members view content ideas from leaders, track their content creation progress, and access content guidance.

### Detailed Implementation Steps

#### 1. Content List Screen
```typescript
// app/(tabs)/content.tsx
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  FlatList,
  TouchableOpacity,
  RefreshControl,
  ActivityIndicator,
} from 'react-native';
import { router } from 'expo-router';
import { SafeAreaView } from 'react-native-safe-area-context';
import { Ionicons } from '@expo/vector-icons';
import { supabase } from '@/lib/supabase';
import { useAuthStore } from '@/stores/authStore';
import { ContentCard } from '@/components/content/ContentCard';
import { EmptyState } from '@/components/common/EmptyState';
import { colors, spacing, typography } from '@/styles/theme';

interface ContentIdea {
  id: string;
  title: string;
  description: string;
  leader_comment: string;
  theme_week: number;
  created_at: string;
  leader: {
    full_name: string;
    role: string;
  };
  submission?: {
    passed: boolean;
    created_at: string;
  };
}

export default function ContentScreen() {
  const { user } = useAuthStore();
  const [contentIdeas, setContentIdeas] = useState<ContentIdea[]>([]);
  const [loading, setLoading] = useState(true);
  const [refreshing, setRefreshing] = useState(false);
  const [filter, setFilter] = useState<'all' | 'pending' | 'completed'>('all');

  useEffect(() => {
    fetchContentIdeas();
    setupRealtimeSubscription();
  }, [filter]);

  const fetchContentIdeas = async () => {
    try {
      let query = supabase
        .from('content_ideas')
        .select(`
          *,
          leader:leader_id(full_name, role),
          submission:content_submissions!left(
            passed,
            created_at
          )
        `)
        .eq('active', true)
        .order('created_at', { ascending: false });

      // Apply filters
      if (filter === 'pending') {
        query = query.is('submission', null);
      } else if (filter === 'completed') {
        query = query.not('submission', 'is', null);
      }

      const { data, error } = await query;

      if (error) throw error;
      
      // Filter to show only content visible to user
      const visibleContent = data?.filter(content => 
        content.leader_id === user?.upline_id || 
        user?.role === 'leader'
      ) || [];

      setContentIdeas(visibleContent);
    } catch (error) {
      console.error('Error fetching content:', error);
    } finally {
      setLoading(false);
      setRefreshing(false);
    }
  };

  const setupRealtimeSubscription = () => {
    const subscription = supabase
      .channel('content_updates')
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          table: 'content_ideas',
        },
        (payload) => {
          // Add new content to list
          fetchContentIdeas();
        }
      )
      .subscribe();

    return () => {
      subscription.unsubscribe();
    };
  };

  const onRefresh = () => {
    setRefreshing(true);
    fetchContentIdeas();
  };

  const getThemeWeekLabel = (week: number) => {
    const themes = [
      'Mindset & Vision',
      'Goal Setting',
      'Time Management',
      'Communication Skills',
      // ... other themes
    ];
    return themes[week % themes.length] || `Week ${week}`;
  };

  const renderHeader = () => (
    <View style={styles.header}>
      <Text style={styles.headerTitle}>Content Ideas</Text>
      <Text style={styles.headerSubtitle}>
        Transform ideas into engaging posts
      </Text>
      
      <View style={styles.filterContainer}>
        <TouchableOpacity
          style={[
            styles.filterButton,
            filter === 'all' && styles.filterButtonActive
          ]}
          onPress={() => setFilter('all')}
        >
          <Text style={[
            styles.filterText,
            filter === 'all' && styles.filterTextActive
          ]}>
            All
          </Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[
            styles.filterButton,
            filter === 'pending' && styles.filterButtonActive
          ]}
          onPress={() => setFilter('pending')}
        >
          <Text style={[
            styles.filterText,
            filter === 'pending' && styles.filterTextActive
          ]}>
            Pending
          </Text>
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[
            styles.filterButton,
            filter === 'completed' && styles.filterButtonActive
          ]}
          onPress={() => setFilter('completed')}
        >
          <Text style={[
            styles.filterText,
            filter === 'completed' && styles.filterTextActive
          ]}>
            Completed
          </Text>
        </TouchableOpacity>
      </View>
    </View>
  );

  const renderItem = ({ item }: { item: ContentIdea }) => (
    <ContentCard
      title={item.title}
      description={item.description}
      leaderComment={item.leader_comment}
      leaderName={item.leader.full_name}
      theme={getThemeWeekLabel(item.theme_week)}
      createdAt={item.created_at}
      isCompleted={!!item.submission}
      isPassed={item.submission?.passed}
      onPress={() => router.push(`/content/${item.id}`)}
    />
  );

  if (loading) {
    return (
      <SafeAreaView style={styles.container}>
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="large" color={colors.primary} />
        </View>
      </SafeAreaView>
    );
  }

  return (
    <SafeAreaView style={styles.container}>
      <FlatList
        data={contentIdeas}
        renderItem={renderItem}
        keyExtractor={(item) => item.id}
        ListHeaderComponent={renderHeader}
        ListEmptyComponent={
          <EmptyState
            icon="document-text"
            title="No content ideas yet"
            description="Check back soon for new content ideas from your leader"
          />
        }
        refreshControl={
          <RefreshControl
            refreshing={refreshing}
            onRefresh={onRefresh}
            tintColor={colors.primary}
          />
        }
        contentContainerStyle={styles.listContent}
      />
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: colors.gray100,
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  header: {
    backgroundColor: colors.white,
    padding: spacing.lg,
    marginBottom: spacing.sm,
  },
  headerTitle: {
    ...typography.h2,
    color: colors.gray900,
    marginBottom: spacing.xs,
  },
  headerSubtitle: {
    ...typography.body,
    color: colors.gray500,
    marginBottom: spacing.lg,
  },
  filterContainer: {
    flexDirection: 'row',
    gap: spacing.sm,
  },
  filterButton: {
    paddingVertical: spacing.sm,
    paddingHorizontal: spacing.md,
    borderRadius: 20,
    backgroundColor: colors.gray100,
  },
  filterButtonActive: {
    backgroundColor: colors.primary,
  },
  filterText: {
    ...typography.caption,
    color: colors.gray700,
    fontFamily: 'Inter_500Medium',
  },
  filterTextActive: {
    color: colors.white,
  },
  listContent: {
    paddingBottom: spacing.xl,
  },
});
```

#### 2. Content Card Component
```typescript
// components/content/ContentCard.tsx
import React from 'react';
import {
  View,
  Text,
  TouchableOpacity,
  StyleSheet,
} from 'react-native';
import { Ionicons } from '@expo/vector-icons';
import { colors, spacing, typography } from '@/styles/theme';
import { formatRelativeTime } from '@/utils/formatters';

interface ContentCardProps {
  title: string;
  description: string;
  leaderComment?: string;
  leaderName: string;
  theme: string;
  createdAt: string;
  isCompleted: boolean;
  isPassed?: boolean;
  onPress: () => void;
}

export const ContentCard: React.FC<ContentCardProps> = ({
  title,
  description,
  leaderComment,
  leaderName,
  theme,
  createdAt,
  isCompleted,
  isPassed,
  onPress,
}) => {
  return (
    <TouchableOpacity
      style={[
        styles.container,
        isCompleted && styles.containerCompleted
      ]}
      onPress={onPress}
      activeOpacity={0.7}
    >
      <View style={styles.header}>
        <View style={styles.themeBadge}>
          <Text style={styles.themeText}>{theme}</Text>
        </View>
        {isCompleted && (
          <View style={[
            styles.statusBadge,
            isPassed ? styles.statusPassed : styles.statusFailed
          ]}>
            <Ionicons
              name={isPassed ? 'checkmark-circle' : 'close-circle'}
              size={16}
              color={colors.white}
            />
            <Text style={styles.statusText}>
              {isPassed ? 'Passed' : 'Failed'}
            </Text>
          </View>
        )}
      </View>

      <Text style={styles.title}>{title}</Text>
      <Text style={styles.description} numberOfLines={2}>
        {description}
      </Text>

      {leaderComment && (
        <View style={styles.commentContainer}>
          <Ionicons name="chatbubble" size={14} color={colors.primary} />
          <Text style={styles.comment} numberOfLines={2}>
            "{leaderComment}"
          </Text>
        </View>
      )}

      <View style={styles.footer}>
        <View style={styles.leaderInfo}>
          <Ionicons name="person" size={14} color={colors.gray500} />
          <Text style={styles.leaderName}>{leaderName}</Text>
        </View>
        <Text style={styles.time}>{formatRelativeTime(createdAt)}</Text>
      </View>

      <Ionicons
        name="chevron-forward"
        size={20}
        color={colors.gray400}
        style={styles.arrow}
      />
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  container: {
    backgroundColor: colors.white,
    marginHorizontal: spacing.md,
    marginVertical: spacing.sm,
    padding: spacing.md,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.05,
    shadowRadius: 4,
    elevation: 2,
  },
  containerCompleted: {
    opacity: 0.8,
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: spacing.sm,
  },
  themeBadge: {
    backgroundColor: colors.primary + '20',
    paddingHorizontal: spacing.sm,
    paddingVertical: spacing.xs,
    borderRadius: 12,
  },
  themeText: {
    ...typography.caption,
    color: colors.primary,
    fontFamily: 'Inter_500Medium',
  },
  statusBadge: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingHorizontal: spacing.sm,
    paddingVertical: spacing.xs,
    borderRadius: 12,
    gap: spacing.xs,
  },
  statusPassed: {
    backgroundColor: colors.success,
  },
  statusFailed: {
    backgroundColor: colors.error,
  },
  statusText: {
    ...typography.caption,
    color: colors.white,
    fontFamily: 'Inter_500Medium',
  },
  title: {
    ...typography.h3,
    color: colors.gray900,
    marginBottom: spacing.xs,
  },
  description: {
    ...typography.body,
    color: colors.gray700,
    marginBottom: spacing.sm,
  },
  commentContainer: {
    flexDirection: 'row',
    alignItems: 'flex-start',
    gap: spacing.xs,
    backgroundColor: colors.gray50,
    padding: spacing.sm,
    borderRadius: 8,
    marginBottom: spacing.sm,
  },
  comment: {
    ...typography.caption,
    color: colors.gray700,
    flex: 1,
    fontStyle: 'italic',
  },
  footer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
  },
  leaderInfo: {
    flexDirection: 'row',
    alignItems: 'center',
    gap: spacing.xs,
  },
  leaderName: {
    ...typography.caption,
    color: colors.gray500,
  },
  time: {
    ...typography.caption,
    color: colors.gray500,
  },
  arrow: {
    position: 'absolute',
    right: spacing.md,
    top: '50%',
  },
});
```

#### 3. Content Detail Screen
```typescript
// app/content/[id].tsx
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  ScrollView,
  TouchableOpacity,
  ActivityIndicator,
  Alert,
} from 'react-native';
import { router, useLocalSearchParams } from 'expo-router';
import { SafeAreaView } from 'react-native-safe-area-context';
import { Ionicons } from '@expo/vector-icons';
import { supabase } from '@/lib/supabase';
import { useAuthStore } from '@/stores/authStore';
import { colors, spacing, typography } from '@/styles/theme';

export default function ContentDetailScreen() {
  const { id } = useLocalSearchParams<{ id: string }>();
  const { user } = useAuthStore();
  const [content, setContent] = useState<any>(null);
  const [loading, setLoading] = useState(true);
  const [submission, setSubmission] = useState<any>(null);

  useEffect(() => {
    fetchContentDetail();
  }, [id]);

  const fetchContentDetail = async () => {
    try {
      const { data: contentData, error: contentError } = await supabase
        .from('content_ideas')
        .select(`
          *,
          leader:leader_id(full_name, role, office:office_id(name))
        `)
        .eq('id', id)
        .single();

      if (contentError) throw contentError;

      // Check if user has already submitted
      const { data: submissionData } = await supabase
        .from('content_submissions')
        .select('*')
        .eq('content_idea_id', id)
        .eq('user_id', user?.id)
        .single();

      setContent(contentData);
      setSubmission(submissionData);
    } catch (error) {
      console.error('Error:', error);
      Alert.alert('Error', 'Failed to load content details');
    } finally {
      setLoading(false);
    }
  };

  const handleStartWriting = () => {
    router.push({
      pathname: '/(tabs)/grade',
      params: { contentIdeaId: id }
    });
  };

  if (loading) {
    return (
      <SafeAreaView style={styles.container}>
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="large" color={colors.primary} />
        </View>
      </SafeAreaView>
    );
  }

  if (!content) {
    return (
      <SafeAreaView style={styles.container}>
        <Text>Content not found</Text>
      </SafeAreaView>
    );
  }

  return (
    <SafeAreaView style={styles.container}>
      <ScrollView showsVerticalScrollIndicator={false}>
        <View style={styles.header}>
          <View style={styles.themeBadge}>
            <Text style={styles.themeText}>Week {content.theme_week}</Text>
          </View>
          
          <Text style={styles.title}>{content.title}</Text>
          
          <View style={styles.leaderInfo}>
            <Ionicons name="person-circle" size={40} color={colors.gray400} />
            <View>
              <Text style={styles.leaderName}>{content.leader.full_name}</Text>
              <Text style={styles.leaderRole}>
                {content.leader.role} • {content.leader.office.name}
              </Text>
            </View>
          </View>
        </View>

        <View style={styles.section}>
          <Text style={styles.sectionTitle}>Content Idea</Text>
          <Text style={styles.description}>{content.description}</Text>
        </View>

        {content.leader_comment && (
          <View style={styles.section}>
            <Text style={styles.sectionTitle}>Leader's Guidance</Text>
            <View style={styles.commentBox}>
              <Ionicons name="chatbubble-ellipses" size={20} color={colors.primary} />
              <Text style={styles.comment}>{content.leader_comment}</Text>
            </View>
          </View>
        )}

        <View style={styles.section}>
          <Text style={styles.sectionTitle}>How to Create Your Post</Text>
          
          <View style={styles.step}>
            <View style={styles.stepNumber}>
              <Text style={styles.stepNumberText}>1</Text>
            </View>
            <View style={styles.stepContent}>
              <Text style={styles.stepTitle}>Choose Your Format</Text>
              <Text style={styles.stepDescription}>
                Pick one: Story, Tip, How-To, or Testimonial
              </Text>
            </View>
          </View>

          <View style={styles.step}>
            <View style={styles.stepNumber}>
              <Text style={styles.stepNumberText}>2</Text>
            </View>
            <View style={styles.stepContent}>
              <Text style={styles.stepTitle}>Write Your Content</Text>
              <Text style={styles.stepDescription}>
                50-300 words with a catchy headline (5-15 words)
              </Text>
            </View>
          </View>

          <View style={styles.step}>
            <View style={styles.stepNumber}>
              <Text style={styles.stepNumberText}>3</Text>
            </View>
            <View style={styles.stepContent}>
              <Text style={styles.stepTitle}>Add Call to Action</Text>
              <Text style={styles.stepDescription}>
                Tell readers how to contact you or take action
              </Text>
            </View>
          </View>

          <View style={styles.step}>
            <View style={styles.stepNumber}>
              <Text style={styles.stepNumberText}>4</Text>
            </View>
            <View style={styles.stepContent}>
              <Text style={styles.stepTitle}>Include Hashtags</Text>
              <Text style={styles.stepDescription}>
                Minimum 3 hashtags including #GalaxyTeam
              </Text>
            </View>
          </View>
        </View>

        {submission ? (
          <View style={styles.submissionStatus}>
            <Ionicons
              name={submission.passed ? 'checkmark-circle' : 'close-circle'}
              size={48}
              color={submission.passed ? colors.success : colors.error}
            />
            <Text style={styles.submissionTitle}>
              Content {submission.passed ? 'Passed' : 'Failed'}
            </Text>
            <Text style={styles.submissionDate}>
              Submitted on {new Date(submission.created_at).toLocaleDateString()}
            </Text>
            {submission.tracking_id && (
              <View style={styles.trackingContainer}>
                <Text style={styles.trackingLabel}>Tracking Link:</Text>
                <Text style={styles.trackingLink}>
                  galaxydreamteam.com/t/{submission.tracking_id}
                </Text>
              </View>
            )}
          </View>
        ) : (
          <TouchableOpacity
            style={styles.startButton}
            onPress={handleStartWriting}
          >
            <Text style={styles.startButtonText}>Start Writing</Text>
            <Ionicons name="arrow-forward" size={20} color={colors.white} />
          </TouchableOpacity>
        )}
      </ScrollView>
    </SafeAreaView>
  );
}

// Styles for content detail screen
```

### Testing Requirements
- [ ] Content list loads correctly
- [ ] Filtering works (all/pending/completed)
- [ ] Pull-to-refresh updates list
- [ ] Content details display properly
- [ ] Navigation to grading works
- [ ] Real-time updates function

### Acceptance Criteria
- Content ideas display with proper filtering
- Leader comments are visible
- Completed status shows correctly
- Navigation flows smoothly
- Real-time updates work without refresh

---

## FRONTEND-005: Content Grading System

**Priority**: High  
**Estimated Hours**: 10  
**Dependencies**: FRONTEND-004

### Objective
Implement the 4-point content grading system with real-time validation, feedback, and tracking link generation.

### Detailed Implementation Steps

#### 1. Grading Screen Implementation
```typescript
// app/(tabs)/grade.tsx
import React, { useState, useEffect, useRef } from 'react';
import {
  View,
  Text,
  TextInput,
  ScrollView,
  TouchableOpacity,
  KeyboardAvoidingView,
  Platform,
  Alert,
  Clipboard,
  ActivityIndicator,
} from 'react-native';
import { router, useLocalSearchParams } from 'expo-router';
import { SafeAreaView } from 'react-native-safe-area-context';
import { Ionicons } from '@expo/vector-icons';
import { supabase } from '@/lib/supabase';
import { useAuthStore } from '@/stores/authStore';
import { validateContent } from '@/utils/contentValidator';
import { ChecklistItem } from '@/components/grade/ChecklistItem';
import { GradeResultModal } from '@/components/grade/GradeResultModal';
import { colors, spacing, typography } from '@/styles/theme';

interface ValidationResult {
  hasHeadline: boolean;
  hasCta: boolean;
  hasBody: boolean;
  hasHashtags: boolean;
  passed: boolean;
  feedback: string[];
}

export default function GradeScreen() {
  const { contentIdeaId } = useLocalSearchParams<{ contentIdeaId?: string }>();
  const { user } = useAuthStore();
  const [content, setContent] = useState('');
  const [validation, setValidation] = useState<ValidationResult | null>(null);
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [showResult, setShowResult] = useState(false);
  const [trackingLink, setTrackingLink] = useState('');
  const [wordCount, setWordCount] = useState(0);
  const [charCount, setCharCount] = useState(0);
  const contentInputRef = useRef<TextInput>(null);

  useEffect(() => {
    if (content) {
      const result = validateContent(content);
      setValidation(result);
      
      // Update counts
      const words = content.trim().split(/\s+/).filter(word => word.length > 0);
      setWordCount(words.length);
      setCharCount(content.length);
    } else {
      setValidation(null);
      setWordCount(0);
      setCharCount(0);
    }
  }, [content]);

  const handleSubmit = async () => {
    if (!validation?.passed) {
      Alert.alert(
        'Content Not Ready',
        'Please fix all issues before submitting.',
        [{ text: 'OK' }]
      );
      return;
    }

    setIsSubmitting(true);
    try {
      // Generate tracking ID
      const { data: seqData } = await supabase
        .rpc('generate_tracking_id');
      
      const trackingId = `GT${String(seqData).padStart(6, '0')}`;

      // Save submission
      const { error } = await supabase
        .from('content_submissions')
        .insert({
          user_id: user?.id,
          content_idea_id: contentIdeaId,
          content_text: content,
          has_headline: validation.hasHeadline,
          has_cta: validation.hasCta,
          has_body: validation.hasBody,
          has_hashtags: validation.hasHashtags,
          passed: true,
          tracking_id: trackingId,
        });

      if (error) throw error;

      // Create analytics entry
      await supabase
        .from('link_analytics')
        .insert({
          tracking_id: trackingId,
          user_id: user?.id,
          total_clicks: 0,
        });

      const link = `https://galaxydreamteam.com/t/${trackingId}`;
      setTrackingLink(link);
      setShowResult(true);

    } catch (error: any) {
      Alert.alert('Error', error.message || 'Failed to submit content');
    } finally {
      setIsSubmitting(false);
    }
  };

  const copyTrackingLink = () => {
    Clipboard.setString(trackingLink);
    Alert.alert('Copied!', 'Tracking link copied to clipboard');
  };

  const resetForm = () => {
    setContent('');
    setValidation(null);
    setShowResult(false);
    setTrackingLink('');
    contentInputRef.current?.focus();
  };

  return (
    <SafeAreaView style={styles.container}>
      <KeyboardAvoidingView
        behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
        style={{ flex: 1 }}
        keyboardVerticalOffset={100}
      >
        <ScrollView
          showsVerticalScrollIndicator={false}
          contentContainerStyle={styles.scrollContent}
        >
          <View style={styles.header}>
            <Text style={styles.title}>Grade Your Content</Text>
            <Text style={styles.subtitle}>
              Make sure your content meets all requirements
            </Text>
          </View>

          <View style={styles.inputSection}>
            <View style={styles.inputHeader}>
              <Text style={styles.inputLabel}>Your Content</Text>
              <View style={styles.counters}>
                <Text style={[
                  styles.counter,
                  wordCount >= 50 && wordCount <= 300 
                    ? styles.counterValid 
                    : styles.counterInvalid
                ]}>
                  {wordCount} words
                </Text>
                <Text style={styles.counterDivider}>•</Text>
                <Text style={styles.counter}>{charCount} chars</Text>
              </View>
            </View>

            <TextInput
              ref={contentInputRef}
              style={styles.textInput}
              multiline
              placeholder="Paste or write your content here..."
              placeholderTextColor={colors.gray500}
              value={content}
              onChangeText={setContent}
              textAlignVertical="top"
            />

            <TouchableOpacity
              style={styles.clearButton}
              onPress={() => setContent('')}
            >
              <Text style={styles.clearButtonText}>Clear</Text>
            </TouchableOpacity>
          </View>

          {validation && (
            <View style={styles.validationSection}>
              <Text style={styles.sectionTitle}>Requirements Check</Text>
              
              <ChecklistItem
                label="Headline (5-15 words)"
                checked={validation.hasHeadline}
                description="First line should grab attention"
              />
              
              <ChecklistItem
                label="Call to Action"
                checked={validation.hasCta}
                description="Include how people can contact you"
              />
              
              <ChecklistItem
                label="Body Content (50-300 words)"
                checked={validation.hasBody}
                description="Tell your story or share value"
              />
              
              <ChecklistItem
                label="Hashtags (3+ including #GalaxyTeam)"
                checked={validation.hasHashtags}
                description="Help people find your content"
              />

              {validation.feedback.length > 0 && (
                <View style={styles.feedbackBox}>
                  <Text style={styles.feedbackTitle}>
                    <Ionicons name="information-circle" size={16} color={colors.warning} />
                    {' '}Improvements Needed:
                  </Text>
                  {validation.feedback.map((item, index) => (
                    <Text key={index} style={styles.feedbackItem}>
                      • {item}
                    </Text>
                  ))}
                </View>
              )}

              <View style={styles.scoreSection}>
                <View style={[
                  styles.scoreCircle,
                  validation.passed ? styles.scorePassed : styles.scoreFailed
                ]}>
                  <Text style={styles.scoreText}>
                    {Object.values(validation).filter(v => v === true).length - 1}/4
                  </Text>
                </View>
                <Text style={[
                  styles.scoreLabel,
                  validation.passed ? styles.scoreLabelPassed : styles.scoreLabelFailed
                ]}>
                  {validation.passed ? 'Ready to Submit!' : 'Not Ready Yet'}
                </Text>
              </View>
            </View>
          )}

          <TouchableOpacity
            style={[
              styles.submitButton,
              (!validation?.passed || isSubmitting) && styles.submitButtonDisabled
            ]}
            onPress={handleSubmit}
            disabled={!validation?.passed || isSubmitting}
          >
            {isSubmitting ? (
              <ActivityIndicator color={colors.white} />
            ) : (
              <>
                <Text style={styles.submitButtonText}>Submit for Tracking Link</Text>
                <Ionicons name="arrow-forward" size={20} color={colors.white} />
              </>
            )}
          </TouchableOpacity>

          <View style={styles.tipsSection}>
            <Text style={styles.tipsTitle}>Quick Tips</Text>
            <View style={styles.tip}>
              <Ionicons name="bulb" size={16} color={colors.primary} />
              <Text style={styles.tipText}>
                Start with a question or bold statement to grab attention
              </Text>
            </View>
            <View style={styles.tip}>
              <Ionicons name="bulb" size={16} color={colors.primary} />
              <Text style={styles.tipText}>
                Share a personal story to connect with your audience
              </Text>
            </View>
            <View style={styles.tip}>
              <Ionicons name="bulb" size={16} color={colors.primary} />
              <Text style={styles.tipText}>
                End with a clear action: "Message me to learn more"
              </Text>
            </View>
          </View>
        </ScrollView>
      </KeyboardAvoidingView>

      <GradeResultModal
        visible={showResult}
        trackingLink={trackingLink}
        onCopyLink={copyTrackingLink}
        onNewContent={resetForm}
        onClose={() => setShowResult(false)}
      />
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: colors.gray50,
  },
  scrollContent: {
    paddingBottom: spacing.xxl,
  },
  header: {
    backgroundColor: colors.white,
    padding: spacing.lg,
    borderBottomWidth: 1,
    borderBottomColor: colors.gray100,
  },
  title: {
    ...typography.h2,
    color: colors.gray900,
    marginBottom: spacing.xs,
  },
  subtitle: {
    ...typography.body,
    color: colors.gray500,
  },
  // ... rest of styles
});
```

#### 2. Content Validator Utility
```typescript
// utils/contentValidator.ts
export interface ValidationResult {
  hasHeadline: boolean;
  hasCta: boolean;
  hasBody: boolean;
  hasHashtags: boolean;
  passed: boolean;
  feedback: string[];
}

export function validateContent(content: string): ValidationResult {
  const lines = content.trim().split('\n');
  const firstLine = lines[0] || '';
  const words = content.split(/\s+/).filter(word => word.length > 0);
  const hashtags = content.match(/#\w+/g) || [];
  
  const result: ValidationResult = {
    hasHeadline: false,
    hasCta: false,
    hasBody: false,
    hasHashtags: false,
    passed: false,
    feedback: [],
  };
  
  // Check 1: Headline (5-15 words in first line)
  const headlineWords = firstLine.split(/\s+/).filter(w => w.length > 0);
  if (headlineWords.length >= 5 && headlineWords.length <= 15) {
    result.hasHeadline = true;
  } else {
    result.feedback.push(
      headlineWords.length < 5 
        ? `Headline too short. You have ${headlineWords.length} words, need at least 5.`
        : `Headline too long. You have ${headlineWords.length} words, maximum is 15.`
    );
  }
  
  // Check 2: Call to Action
  const ctaKeywords = [
    // English
    'contact', 'call', 'whatsapp', 'message', 'dm', 'inbox',
    'click', 'join', 'register', 'sign up', 'learn more',
    'get started', 'reach out', 'connect', 'follow',
    // Amharic
    'ደውሉ', 'መልእክት', 'ይቀላቀሉ', 'ያግኙን', 'ለመረጃ'
  ];
  
  const contentLower = content.toLowerCase();
  const hasCta = ctaKeywords.some(keyword => contentLower.includes(keyword));
  
  if (hasCta) {
    result.hasCta = true;
  } else {
    result.feedback.push(
      'Missing call to action. Add how people can contact or engage with you.'
    );
  }
  
  // Check 3: Body (50-300 words)
  if (words.length >= 50 && words.length <= 300) {
    result.hasBody = true;
  } else {
    result.feedback.push(
      words.length < 50
        ? `Content too short. You have ${words.length} words, need at least 50.`
        : `Content too long. You have ${words.length} words, maximum is 300.`
    );
  }
  
  // Check 4: Hashtags (minimum 3, must include #GalaxyTeam)
  const hasGalaxyTag = hashtags.some(tag => 
    tag.toLowerCase() === '#galaxyteam'
  );
  
  if (hashtags.length >= 3 && hasGalaxyTag) {
    result.hasHashtags = true;
  } else {
    if (!hasGalaxyTag) {
      result.feedback.push('Must include #GalaxyTeam hashtag');
    }
    if (hashtags.length < 3) {
      result.feedback.push(
        `Need at least 3 hashtags. You have ${hashtags.length}.`
      );
    }
  }
  
  // Overall pass/fail
  result.passed = result.hasHeadline && result.hasCta && 
                  result.hasBody && result.hasHashtags;
  
  return result;
}

// Helper function to extract hashtags
export function extractHashtags(content: string): string[] {
  const hashtags = content.match(/#\w+/g) || [];
  return [...new Set(hashtags)]; // Remove duplicates
}

// Helper function to count words
export function countWords(content: string): number {
  return content.trim().split(/\s+/).filter(word => word.length > 0).length;
}

// Helper function to suggest improvements
export function getSuggestions(validation: ValidationResult): string[] {
  const suggestions = [];
  
  if (!validation.hasHeadline) {
    suggestions.push('Try starting with a question or bold statement');
  }
  
  if (!validation.hasCta) {
    suggestions.push('Add "Message me to learn more" or similar');
  }
  
  if (!validation.hasBody) {
    suggestions.push('Share a personal story or valuable tip');
  }
  
  if (!validation.hasHashtags) {
    suggestions.push('Use hashtags like #GalaxyTeam #Success #Ethiopia');
  }
  
  return suggestions;
}
```

#### 3. Checklist Item Component
```typescript
// components/grade/ChecklistItem.tsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { Ionicons } from '@expo/vector-icons';
import { colors, spacing, typography } from '@/styles/theme';

interface ChecklistItemProps {
  label: string;
  checked: boolean;
  description?: string;
}

export const ChecklistItem: React.FC<ChecklistItemProps> = ({
  label,
  checked,
  description,
}) => {
  return (
    <View style={styles.container}>
      <View style={styles.mainRow}>
        <View style={[styles.checkbox, checked && styles.checkboxChecked]}>
          {checked && (
            <Ionicons name="checkmark" size={16} color={colors.white} />
          )}
        </View>
        <Text style={[styles.label, checked && styles.labelChecked]}>
          {label}
        </Text>
      </View>
      {description && (
        <Text style={styles.description}>{description}</Text>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    paddingVertical: spacing.md,
    borderBottomWidth: 1,
    borderBottomColor: colors.gray100,
  },
  mainRow: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  checkbox: {
    width: 24,
    height: 24,
    borderRadius: 12,
    borderWidth: 2,
    borderColor: colors.gray300,
    alignItems: 'center',
    justifyContent: 'center',
    marginRight: spacing.md,
  },
  checkboxChecked: {
    backgroundColor: colors.success,
    borderColor: colors.success,
  },
  label: {
    ...typography.body,
    color: colors.gray700,
    flex: 1,
  },
  labelChecked: {
    color: colors.gray900,
    fontFamily: 'Inter_500Medium',
  },
  description: {
    ...typography.caption,
    color: colors.gray500,
    marginLeft: 40,
    marginTop: spacing.xs,
  },
});
```

### Testing Requirements
- [ ] Content validation works correctly
- [ ] Real-time feedback updates
- [ ] Word count is accurate
- [ ] Tracking ID generation works
- [ ] Copy link functionality works

### Acceptance Criteria
- All 4 validation rules work correctly
- Feedback is specific and helpful
- Tracking links generate properly
- Copy to clipboard works
- Form resets after submission

---

## FRONTEND-006: Analytics Dashboard

**Priority**: High  
**Estimated Hours**: 10  
**Dependencies**: FRONTEND-005

### Objective
Create a simple analytics display showing click counts, best performing posts, and lead notifications with real-time updates.

### Detailed Implementation Steps

#### 1. Analytics Screen
```typescript
// app/(tabs)/analytics.tsx
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  ScrollView,
  TouchableOpacity,
  RefreshControl,
  ActivityIndicator,
} from 'react-native';
import { router } from 'expo-router';
import { SafeAreaView } from 'react-native-safe-area-context';
import { Ionicons } from '@expo/vector-icons';
import { LinearGradient } from 'expo-linear-gradient';
import { supabase } from '@/lib/supabase';
import { useAuthStore } from '@/stores/authStore';
import { SimpleChart } from '@/components/analytics/SimpleChart';
import { MetricCard } from '@/components/analytics/MetricCard';
import { TopPostCard } from '@/components/analytics/TopPostCard';
import { colors, spacing, typography } from '@/styles/theme';

interface AnalyticsData {
  todayClicks: number;
  yesterdayClicks: number;
  weekClicks: number;
  monthClicks: number;
  totalClicks: number;
  topPost: any;
  recentLeads: any[];
  clickHistory: any[];
}

export default function AnalyticsScreen() {
  const { user } = useAuthStore();
  const [analytics, setAnalytics] = useState<AnalyticsData>({
    todayClicks: 0,
    yesterdayClicks: 0,
    weekClicks: 0,
    monthClicks: 0,
    totalClicks: 0,
    topPost: null,
    recentLeads: [],
    clickHistory: [],
  });
  const [loading, setLoading] = useState(true);
  const [refreshing, setRefreshing] = useState(false);
  const [timeRange, setTimeRange] = useState<'week' | 'month'>('week');

  useEffect(() => {
    fetchAnalytics();
    setupRealtimeSubscription();
  }, [timeRange]);

  const fetchAnalytics = async () => {
    try {
      const now = new Date();
      const today = now.toISOString().split('T')[0];
      
      const yesterday = new Date(now);
      yesterday.setDate(yesterday.getDate() - 1);
      const yesterdayStr = yesterday.toISOString().split('T')[0];
      
      const weekAgo = new Date(now);
      weekAgo.setDate(weekAgo.getDate() - 7);
      
      const monthAgo = new Date(now);
      monthAgo.setMonth(monthAgo.getMonth() - 1);

      // Fetch all user's analytics
      const { data: allAnalytics } = await supabase
        .from('link_analytics')
        .select(`
          *,
          submission:content_submissions!tracking_id(
            content_text,
            created_at
          )
        `)
        .eq('user_id', user?.id)
        .order('total_clicks', { ascending: false });

      if (!allAnalytics) return;

      // Calculate metrics
      let todayClicks = 0;
      let yesterdayClicks = 0;
      let weekClicks = 0;
      let monthClicks = 0;
      let totalClicks = 0;

      allAnalytics.forEach(item => {
        totalClicks += item.total_clicks;
        
        if (item.last_clicked) {
          const clickDate = new Date(item.last_clicked);
          
          if (clickDate.toISOString().split('T')[0] === today) {
            todayClicks += item.total_clicks;
          }
          if (clickDate.toISOString().split('T')[0] === yesterdayStr) {
            yesterdayClicks += item.total_clicks;
          }
          if (clickDate >= weekAgo) {
            weekClicks += item.total_clicks;
          }
          if (clickDate >= monthAgo) {
            monthClicks += item.total_clicks;
          }
        }
      });

      // Get top performing post
      const topPost = allAnalytics[0];

      // Get recent leads (clicks from today)
      const { data: recentLeads } = await supabase
        .from('lead_notifications')
        .select('*')
        .eq('user_id', user?.id)
        .gte('clicked_at', today)
        .order('clicked_at', { ascending: false })
        .limit(10);

      // Get click history for chart
      const { data: clickHistory } = await supabase
        .from('link_clicks')
        .select('clicked_at, tracking_id')
        .in('tracking_id', allAnalytics.map(a => a.tracking_id))
        .gte('clicked_at', timeRange === 'week' ? weekAgo.toISOString() : monthAgo.toISOString())
        .order('clicked_at', { ascending: true });

      setAnalytics({
        todayClicks,
        yesterdayClicks,
        weekClicks,
        monthClicks,
        totalClicks,
        topPost,
        recentLeads: recentLeads || [],
        clickHistory: clickHistory || [],
      });
    } catch (error) {
      console.error('Error fetching analytics:', error);
    } finally {
      setLoading(false);
      setRefreshing(false);
    }
  };

  const setupRealtimeSubscription = () => {
    const subscription = supabase
      .channel('analytics_updates')
      .on(
        'postgres_changes',
        {
          event: 'UPDATE',
          schema: 'public',
          table: 'link_analytics',
          filter: `user_id=eq.${user?.id}`,
        },
        () => {
          fetchAnalytics();
        }
      )
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          table: 'lead_notifications',
          filter: `user_id=eq.${user?.id}`,
        },
        (payload) => {
          // Show notification
          showLeadNotification(payload.new);
          fetchAnalytics();
        }
      )
      .subscribe();

    return () => {
      subscription.unsubscribe();
    };
  };

  const showLeadNotification = (lead: any) => {
    // This would trigger a push notification in production
    console.log('New lead:', lead);
  };

  const onRefresh = () => {
    setRefreshing(true);
    fetchAnalytics();
  };

  const getTodayChange = () => {
    if (analytics.yesterdayClicks === 0) return 0;
    return ((analytics.todayClicks - analytics.yesterdayClicks) / analytics.yesterdayClicks * 100).toFixed(0);
  };

  if (loading) {
    return (
      <SafeAreaView style={styles.container}>
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="large" color={colors.primary} />
        </View>
      </SafeAreaView>
    );
  }

  return (
    <SafeAreaView style={styles.container}>
      <ScrollView
        showsVerticalScrollIndicator={false}
        refreshControl={
          <RefreshControl
            refreshing={refreshing}
            onRefresh={onRefresh}
            tintColor={colors.primary}
          />
        }
      >
        {/* Header Stats */}
        <LinearGradient
          colors={[colors.primary, colors.primary + 'DD']}
          style={styles.headerGradient}
        >
          <Text style={styles.headerTitle}>Your Analytics</Text>
          <View style={styles.mainMetric}>
            <Text style={styles.mainMetricValue}>
              {analytics.todayClicks}
            </Text>
            <Text style={styles.mainMetricLabel}>Clicks Today</Text>
            {analytics.yesterdayClicks > 0 && (
              <View style={styles.changeIndicator}>
                <Ionicons
                  name={Number(getTodayChange()) >= 0 ? 'trending-up' : 'trending-down'}
                  size={16}
                  color={colors.white}
                />
                <Text style={styles.changeText}>
                  {getTodayChange()}% from yesterday
                </Text>
              </View>
            )}
          </View>
        </LinearGradient>

        {/* Quick Metrics */}
        <View style={styles.metricsGrid}>
          <MetricCard
            title="This Week"
            value={analytics.weekClicks}
            icon="calendar"
            color={colors.secondary}
          />
          <MetricCard
            title="This Month"
            value={analytics.monthClicks}
            icon="calendar-outline"
            color={colors.accent}
          />
          <MetricCard
            title="All Time"
            value={analytics.totalClicks}
            icon="infinite"
            color={colors.primary}
          />
          <MetricCard
            title="Today's Leads"
            value={analytics.recentLeads.length}
            icon="people"
            color={colors.success}
          />
        </View>

        {/* Click Trend Chart */}
        <View style={styles.chartSection}>
          <View style={styles.chartHeader}>
            <Text style={styles.sectionTitle}>Click Trends</Text>
            <View style={styles.timeRangeSelector}>
              <TouchableOpacity
                style={[
                  styles.timeRangeButton,
                  timeRange === 'week' && styles.timeRangeButtonActive
                ]}
                onPress={() => setTimeRange('week')}
              >
                <Text style={[
                  styles.timeRangeText,
                  timeRange === 'week' && styles.timeRangeTextActive
                ]}>
                  Week
                </Text>
              </TouchableOpacity>
              <TouchableOpacity
                style={[
                  styles.timeRangeButton,
                  timeRange === 'month' && styles.timeRangeButtonActive
                ]}
                onPress={() => setTimeRange('month')}
              >
                <Text style={[
                  styles.timeRangeText,
                  timeRange === 'month' && styles.timeRangeTextActive
                ]}>
                  Month
                </Text>
              </TouchableOpacity>
            </View>
          </View>
          
          <SimpleChart
            data={analytics.clickHistory}
            timeRange={timeRange}
          />
        </View>

        {/* Top Performing Post */}
        {analytics.topPost && (
          <View style={styles.section}>
            <Text style={styles.sectionTitle}>Best Performing Post</Text>
            <TopPostCard
              post={analytics.topPost}
              onPress={() => router.push(`/analytics/post/${analytics.topPost.tracking_id}`)}
            />
          </View>
        )}

        {/* Recent Leads */}
        {analytics.recentLeads.length > 0 && (
          <View style={styles.section}>
            <Text style={styles.sectionTitle}>Today's Leads</Text>
            <View style={styles.leadsList}>
              {analytics.recentLeads.map((lead, index) => (
                <View key={lead.id} style={styles.leadItem}>
                  <View style={styles.leadIcon}>
                    <Ionicons name="person" size={16} color={colors.primary} />
                  </View>
                  <Text style={styles.leadTime}>
                    {new Date(lead.clicked_at).toLocaleTimeString()}
                  </Text>
                  <Text style={styles.leadTracking}>
                    via {lead.tracking_id}
                  </Text>
                </View>
              ))}
            </View>
          </View>
        )}

        {/* CTA to ERP */}
        <TouchableOpacity
          style={styles.erpButton}
          onPress={() => {
            // Open ERP dashboard in browser
            Alert.alert(
              'Open ERP Dashboard',
              'View detailed analytics in your ERP system?',
              [
                { text: 'Cancel', style: 'cancel' },
                { text: 'Open', onPress: () => {/* Open URL */} }
              ]
            );
          }}
        >
          <Ionicons name="bar-chart" size={20} color={colors.primary} />
          <Text style={styles.erpButtonText}>
            View Detailed Analytics in ERP
          </Text>
          <Ionicons name="arrow-forward" size={16} color={colors.primary} />
        </TouchableOpacity>
      </ScrollView>
    </SafeAreaView>
  );
}

// Styles for analytics screen
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: colors.gray50,
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  headerGradient: {
    paddingTop: spacing.xl,
    paddingBottom: spacing.xxl,
    paddingHorizontal: spacing.lg,
  },
  headerTitle: {
    ...typography.h2,
    color: colors.white,
    marginBottom: spacing.lg,
  },
  mainMetric: {
    alignItems: 'center',
  },
  mainMetricValue: {
    fontSize: 48,
    fontFamily: 'Inter_700Bold',
    color: colors.white,
    marginBottom: spacing.xs,
  },
  mainMetricLabel: {
    ...typography.body,
    color: colors.white,
    opacity: 0.9,
  },
  changeIndicator: {
    flexDirection: 'row',
    alignItems: 'center',
    marginTop: spacing.sm,
    gap: spacing.xs,
  },
  changeText: {
    ...typography.caption,
    color: colors.white,
  },
  metricsGrid: {
    flexDirection: 'row',
    flexWrap: 'wrap',
    padding: spacing.md,
    gap: spacing.md,
  },
  chartSection: {
    backgroundColor: colors.white,
    margin: spacing.md,
    padding: spacing.lg,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.05,
    shadowRadius: 4,
    elevation: 2,
  },
  chartHeader: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: spacing.lg,
  },
  sectionTitle: {
    ...typography.h3,
    color: colors.gray900,
  },
  timeRangeSelector: {
    flexDirection: 'row',
    backgroundColor: colors.gray100,
    borderRadius: 8,
    padding: 2,
  },
  timeRangeButton: {
    paddingVertical: spacing.sm,
    paddingHorizontal: spacing.md,
    borderRadius: 6,
  },
  timeRangeButtonActive: {
    backgroundColor: colors.white,
  },
  timeRangeText: {
    ...typography.caption,
    color: colors.gray600,
    fontFamily: 'Inter_500Medium',
  },
  timeRangeTextActive: {
    color: colors.gray900,
  },
  section: {
    padding: spacing.md,
  },
  leadsList: {
    backgroundColor: colors.white,
    borderRadius: 12,
    padding: spacing.md,
    marginTop: spacing.sm,
  },
  leadItem: {
    flexDirection: 'row',
    alignItems: 'center',
    paddingVertical: spacing.sm,
    borderBottomWidth: 1,
    borderBottomColor: colors.gray100,
  },
  leadIcon: {
    width: 32,
    height: 32,
    borderRadius: 16,
    backgroundColor: colors.primary + '10',
    alignItems: 'center',
    justifyContent: 'center',
    marginRight: spacing.md,
  },
  leadTime: {
    ...typography.body,
    color: colors.gray900,
    flex: 1,
  },
  leadTracking: {
    ...typography.caption,
    color: colors.gray500,
  },
  erpButton: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    backgroundColor: colors.white,
    margin: spacing.md,
    padding: spacing.lg,
    borderRadius: 12,
    borderWidth: 1,
    borderColor: colors.primary,
    gap: spacing.sm,
  },
  erpButtonText: {
    ...typography.body,
    color: colors.primary,
    fontFamily: 'Inter_600SemiBold',
    flex: 1,
    textAlign: 'center',
  },
});
```

#### 2. Analytics Components

```typescript
// components/analytics/MetricCard.tsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { Ionicons } from '@expo/vector-icons';
import { colors, spacing, typography } from '@/styles/theme';
import { formatNumber } from '@/utils/formatters';

interface MetricCardProps {
  title: string;
  value: number;
  icon: keyof typeof Ionicons.glyphMap;
  color: string;
}

export const MetricCard: React.FC<MetricCardProps> = ({
  title,
  value,
  icon,
  color,
}) => {
  return (
    <View style={styles.container}>
      <View style={[styles.iconContainer, { backgroundColor: color + '20' }]}>
        <Ionicons name={icon} size={24} color={color} />
      </View>
      <Text style={styles.value}>{formatNumber(value)}</Text>
      <Text style={styles.title}>{title}</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    backgroundColor: colors.white,
    borderRadius: 12,
    padding: spacing.md,
    alignItems: 'center',
    flex: 1,
    minWidth: '45%',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.05,
    shadowRadius: 4,
    elevation: 2,
  },
  iconContainer: {
    width: 48,
    height: 48,
    borderRadius: 24,
    alignItems: 'center',
    justifyContent: 'center',
    marginBottom: spacing.sm,
  },
  value: {
    ...typography.h2,
    color: colors.gray900,
    marginBottom: spacing.xs,
  },
  title: {
    ...typography.caption,
    color: colors.gray500,
    textAlign: 'center',
  },
});

// components/analytics/SimpleChart.tsx
import React from 'react';
import { View, Text, StyleSheet, Dimensions } from 'react-native';
import Svg, { Line, Polyline, Circle, Text as SvgText } from 'react-native-svg';
import { colors, spacing, typography } from '@/styles/theme';

interface ChartData {
  clicked_at: string;
  tracking_id: string;
}

interface SimpleChartProps {
  data: ChartData[];
  timeRange: 'week' | 'month';
}

export const SimpleChart: React.FC<SimpleChartProps> = ({ data, timeRange }) => {
  const screenWidth = Dimensions.get('window').width - (spacing.md * 4);
  const chartHeight = 200;
  const padding = 20;

  // Group data by day
  const groupedData = data.reduce((acc, item) => {
    const date = new Date(item.clicked_at).toISOString().split('T')[0];
    acc[date] = (acc[date] || 0) + 1;
    return acc;
  }, {} as Record<string, number>);

  // Create array of days
  const days = timeRange === 'week' ? 7 : 30;
  const endDate = new Date();
  const dataPoints = [];

  for (let i = days - 1; i >= 0; i--) {
    const date = new Date(endDate);
    date.setDate(date.getDate() - i);
    const dateStr = date.toISOString().split('T')[0];
    dataPoints.push({
      date: dateStr,
      value: groupedData[dateStr] || 0,
      label: date.getDate().toString(),
    });
  }

  // Calculate chart points
  const maxValue = Math.max(...dataPoints.map(d => d.value), 1);
  const xStep = (screenWidth - padding * 2) / (dataPoints.length - 1);
  const yScale = (chartHeight - padding * 2) / maxValue;

  const points = dataPoints.map((point, index) => ({
    x: padding + index * xStep,
    y: chartHeight - padding - (point.value * yScale),
    value: point.value,
    label: point.label,
  }));

  const polylinePoints = points.map(p => `${p.x},${p.y}`).join(' ');

  return (
    <View style={styles.container}>
      <Svg width={screenWidth} height={chartHeight}>
        {/* Grid lines */}
        {[0, 1, 2, 3, 4].map(i => (
          <Line
            key={i}
            x1={padding}
            y1={padding + i * (chartHeight - padding * 2) / 4}
            x2={screenWidth - padding}
            y2={padding + i * (chartHeight - padding * 2) / 4}
            stroke={colors.gray200}
            strokeWidth="1"
          />
        ))}

        {/* Chart line */}
        <Polyline
          points={polylinePoints}
          fill="none"
          stroke={colors.primary}
          strokeWidth="2"
        />

        {/* Data points */}
        {points.map((point, index) => (
          <Circle
            key={index}
            cx={point.x}
            cy={point.y}
            r="4"
            fill={colors.primary}
          />
        ))}

        {/* Labels */}
        {points.filter((_, i) => i % Math.ceil(points.length / 7) === 0).map((point, index) => (
          <SvgText
            key={index}
            x={point.x}
            y={chartHeight - 5}
            fill={colors.gray500}
            fontSize="12"
            textAnchor="middle"
          >
            {point.label}
          </SvgText>
        ))}
      </Svg>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    marginTop: spacing.sm,
  },
});

// components/analytics/TopPostCard.tsx
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { Ionicons } from '@expo/vector-icons';
import { colors, spacing, typography } from '@/styles/theme';

interface TopPostCardProps {
  post: {
    tracking_id: string;
    total_clicks: number;
    submission?: {
      content_text: string;
      created_at: string;
    };
  };
  onPress: () => void;
}

export const TopPostCard: React.FC<TopPostCardProps> = ({ post, onPress }) => {
  if (!post.submission) return null;

  return (
    <TouchableOpacity style={styles.container} onPress={onPress}>
      <View style={styles.header}>
        <View style={styles.statsContainer}>
          <Ionicons name="trending-up" size={20} color={colors.success} />
          <Text style={styles.clickCount}>{post.total_clicks} clicks</Text>
        </View>
        <Text style={styles.trackingId}>{post.tracking_id}</Text>
      </View>
      
      <Text style={styles.content} numberOfLines={3}>
        {post.submission.content_text}
      </Text>
      
      <View style={styles.footer}>
        <Text style={styles.date}>
          Posted {new Date(post.submission.created_at).toLocaleDateString()}
        </Text>
        <Ionicons name="chevron-forward" size={20} color={colors.gray400} />
      </View>
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  container: {
    backgroundColor: colors.white,
    borderRadius: 12,
    padding: spacing.md,
    marginTop: spacing.sm,
    borderWidth: 1,
    borderColor: colors.success + '30',
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    marginBottom: spacing.sm,
  },
  statsContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    gap: spacing.xs,
  },
  clickCount: {
    ...typography.body,
    color: colors.success,
    fontFamily: 'Inter_600SemiBold',
  },
  trackingId: {
    ...typography.caption,
    color: colors.gray500,
  },
  content: {
    ...typography.body,
    color: colors.gray700,
    marginBottom: spacing.sm,
  },
  footer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
  },
  date: {
    ...typography.caption,
    color: colors.gray500,
  },
});
```

### Testing Requirements
- [ ] Analytics data loads correctly
- [ ] Real-time updates work
- [ ] Chart displays properly
- [ ] Lead notifications function
- [ ] Time range switching works

### Acceptance Criteria
- Click counts are accurate and real-time
- Chart visualization works smoothly
- Lead notifications trigger properly
- Navigation to ERP works
- Performance is optimized for large datasets

---

## FRONTEND-007: Messaging & Communication

**Priority**: Medium  
**Estimated Hours**: 12  
**Dependencies**: FRONTEND-003

### Objective
Implement messaging system for upline communication, announcements viewing, and push notifications.

### Detailed Implementation Steps

#### 1. Messages List Screen
```typescript
// app/(tabs)/messages.tsx
import React, { useState, useEffect } from 'react';
import {
  View,
  Text,
  FlatList,
  TouchableOpacity,
  RefreshControl,
  ActivityIndicator,
} from 'react-native';
import { router } from 'expo-router';
import { SafeAreaView } from 'react-native-safe-area-context';
import { Ionicons } from '@expo/vector-icons';
import { supabase } from '@/lib/supabase';
import { useAuthStore } from '@/stores/authStore';
import { ConversationItem } from '@/components/messages/ConversationItem';
import { AnnouncementCard } from '@/components/messages/AnnouncementCard';
import { EmptyState } from '@/components/common/EmptyState';
import { colors, spacing, typography } from '@/styles/theme';

export default function MessagesScreen() {
  const { user } = useAuthStore();
  const [activeTab, setActiveTab] = useState<'messages' | 'announcements'>('messages');
  const [conversations, setConversations] = useState([]);
  const [announcements, setAnnouncements] = useState([]);
  const [loading, setLoading] = useState(true);
  const [refreshing, setRefreshing] = useState(false);

  useEffect(() => {
    if (activeTab === 'messages') {
      fetchConversations();
    } else {
      fetchAnnouncements();
    }
    setupRealtimeSubscription();
  }, [activeTab]);

  const fetchConversations = async () => {
    try {
      // Get all messages for user
      const { data: messages } = await supabase
        .from('messages')
        .select(`
          *,
          sender:sender_id(id, full_name, role),
          recipient:recipient_id(id, full_name, role)
        `)
        .or(`sender_id.eq.${user?.id},recipient_id.eq.${user?.id}`)
        .order('created_at', { ascending: false });

      // Group by conversation
      const grouped = groupMessagesByConversation(messages || []);
      setConversations(grouped);
    } catch (error) {
      console.error('Error fetching conversations:', error);
    } finally {
      setLoading(false);
      setRefreshing(false);
    }
  };

  const fetchAnnouncements = async () => {
    try {
      const { data } = await supabase
        .from('announcements')
        .select(`
          *,
          author:author_id(full_name, role)
        `)
        .or(`target_office_id.eq.${user?.office_id},target_office_id.is.null`)
        .order('created_at', { ascending: false })
        .limit(20);

      setAnnouncements(data || []);
    } catch (error) {
      console.error('Error fetching announcements:', error);
    } finally {
      setLoading(false);
      setRefreshing(false);
    }
  };

  const groupMessagesByConversation = (messages: any[]) => {
    const conversations = new Map();

    messages.forEach(message => {
      const otherUserId = message.sender_id === user?.id 
        ? message.recipient_id 
        : message.sender_id;
      
      const otherUser = message.sender_id === user?.id 
        ? message.recipient 
        : message.sender;

      if (!conversations.has(otherUserId)) {
        conversations.set(otherUserId, {
          otherUser,
          lastMessage: message,
          unreadCount: 0,
        });
      }

      // Count unread messages
      if (message.recipient_id === user?.id && !message.read) {
        const conv = conversations.get(otherUserId);
        conv.unreadCount++;
      }
    });

    return Array.from(conversations.values());
  };

  const setupRealtimeSubscription = () => {
    const subscription = supabase
      .channel('messages_updates')
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          table: 'messages',
          filter: `recipient_id=eq.${user?.id}`,
        },
        () => {
          fetchConversations();
        }
      )
      .subscribe();

    return () => {
      subscription.unsubscribe();
    };
  };

  const markAsRead = async (conversationUserId: string) => {
    await supabase
      .from('messages')
      .update({ read: true })
      .eq('sender_id', conversationUserId)
      .eq('recipient_id', user?.id)
      .eq('read', false);
  };

  const onRefresh = () => {
    setRefreshing(true);
    if (activeTab === 'messages') {
      fetchConversations();
    } else {
      fetchAnnouncements();
    }
  };

  const renderConversation = ({ item }: any) => (
    <ConversationItem
      otherUser={item.otherUser}
      lastMessage={item.lastMessage}
      unreadCount={item.unreadCount}
      onPress={() => {
        markAsRead(item.otherUser.id);
        router.push(`/chat/${item.otherUser.id}`);
      }}
    />
  );

  const renderAnnouncement = ({ item }: any) => (
    <AnnouncementCard
      title={item.title}
      content={item.content}
      author={item.author.full_name}
      createdAt={item.created_at}
      onPress={() => router.push(`/announcement/${item.id}`)}
    />
  );

  return (
    <SafeAreaView style={styles.container}>
      {/* Tab Selector */}
      <View style={styles.tabContainer}>
        <TouchableOpacity
          style={[
            styles.tab,
            activeTab === 'messages' && styles.tabActive
          ]}
          onPress={() => setActiveTab('messages')}
        >
          <Text style={[
            styles.tabText,
            activeTab === 'messages' && styles.tabTextActive
          ]}>
            Messages
          </Text>
          {conversations.some(c => c.unreadCount > 0) && (
            <View style={styles.badge} />
          )}
        </TouchableOpacity>
        
        <TouchableOpacity
          style={[
            styles.tab,
            activeTab === 'announcements' && styles.tabActive
          ]}
          onPress={() => setActiveTab('announcements')}
        >
          <Text style={[
            styles.tabText,
            activeTab === 'announcements' && styles.tabTextActive
          ]}>
            Announcements
          </Text>
        </TouchableOpacity>
      </View>

      {/* Content */}
      {loading ? (
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="large" color={colors.primary} />
        </View>
      ) : (
        <FlatList
          data={activeTab === 'messages' ? conversations : announcements}
          renderItem={activeTab === 'messages' ? renderConversation : renderAnnouncement}
          keyExtractor={(item) => 
            activeTab === 'messages' 
              ? item.otherUser.id 
              : item.id
          }
          ListEmptyComponent={
            <EmptyState
              icon={activeTab === 'messages' ? 'chatbubbles' : 'megaphone'}
              title={activeTab === 'messages' ? 'No messages yet' : 'No announcements'}
              description={
                activeTab === 'messages' 
                  ? 'Start a conversation with your upline'
                  : 'Check back for important updates'
              }
              action={
                activeTab === 'messages' && user?.upline_id
                  ? {
                      label: 'Message Upline',
                      onPress: () => router.push(`/chat/${user.upline_id}`)
                    }
                  : undefined
              }
            />
          }
          refreshControl={
            <RefreshControl
              refreshing={refreshing}
              onRefresh={onRefresh}
              tintColor={colors.primary}
            />
          }
          contentContainerStyle={styles.listContent}
        />
      )}

      {/* FAB for new message */}
      {activeTab === 'messages' && user?.upline_id && (
        <TouchableOpacity
          style={styles.fab}
          onPress={() => router.push(`/chat/${user.upline_id}`)}
        >
          <Ionicons name="create" size={24} color={colors.white} />
        </TouchableOpacity>
      )}
    </SafeAreaView>
  );
}

// Styles for messages screen
```

#### 2. Chat Screen Implementation
```typescript
// app/chat/[userId].tsx
import React, { useState, useEffect, useRef } from 'react';
import {
  View,
  Text,
  TextInput,
  FlatList,
  TouchableOpacity,
  KeyboardAvoidingView,
  Platform,
  ActivityIndicator,
} from 'react-native';
import { router, useLocalSearchParams } from 'expo-router';
import { SafeAreaView } from 'react-native-safe-area-context';
import { Ionicons } from '@expo/vector-icons';
import { supabase } from '@/lib/supabase';
import { useAuthStore } from '@/stores/authStore';
import { MessageBubble } from '@/components/messages/MessageBubble';
import { colors, spacing, typography } from '@/styles/theme';

export default function ChatScreen() {
  const { userId } = useLocalSearchParams<{ userId: string }>();
  const { user } = useAuthStore();
  const [messages, setMessages] = useState([]);
  const [newMessage, setNewMessage] = useState('');
  const [otherUser, setOtherUser] = useState<any>(null);
  const [loading, setLoading] = useState(true);
  const [sending, setSending] = useState(false);
  const flatListRef = useRef<FlatList>(null);

  useEffect(() => {
    fetchUserAndMessages();
    setupRealtimeSubscription();
    markMessagesAsRead();
  }, [userId]);

  const fetchUserAndMessages = async () => {
    try {
      // Fetch other user details
      const { data: userData } = await supabase
        .from('profiles')
        .select('*')
        .eq('id', userId)
        .single();

      setOtherUser(userData);

      // Fetch messages
      const { data: messagesData } = await supabase
        .from('messages')
        .select('*')
        .or(`sender_id.eq.${user?.id},recipient_id.eq.${user?.id}`)
        .or(`sender_id.eq.${userId},recipient_id.eq.${userId}`)
        .order('created_at', { ascending: true });

      setMessages(messagesData || []);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
    }
  };

  const setupRealtimeSubscription = () => {
    const subscription = supabase
      .channel(`chat_${userId}`)
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          table: 'messages',
          filter: `sender_id=eq.${userId}`,
        },
        (payload) => {
          if (payload.new.recipient_id === user?.id) {
            setMessages(prev => [...prev, payload.new]);
            markMessagesAsRead();
          }
        }
      )
      .subscribe();

    return () => {
      subscription.unsubscribe();
    };
  };

  const markMessagesAsRead = async () => {
    await supabase
      .from('messages')
      .update({ read: true })
      .eq('sender_id', userId)
      .eq('recipient_id', user?.id)
      .eq('read', false);
  };

  const sendMessage = async () => {
    if (!newMessage.trim() || sending) return;

    setSending(true);
    const messageText = newMessage.trim();
    setNewMessage('');

    try {
      const { data, error } = await supabase
        .from('messages')
        .insert({
          sender_id: user?.id,
          recipient_id: userId,
          message: messageText,
        })
        .select()
        .single();

      if (error) throw error;

      setMessages(prev => [...prev, data]);
    } catch (error) {
      console.error('Error sending message:', error);
      setNewMessage(messageText); // Restore message on error
    } finally {
      setSending(false);
    }
  };

  if (loading) {
    return (
      <SafeAreaView style={styles.container}>
        <View style={styles.loadingContainer}>
          <ActivityIndicator size="large" color={colors.primary} />
        </View>
      </SafeAreaView>
    );
  }

  return (
    <SafeAreaView style={styles.container}>
      {/* Chat Header */}
      <View style={styles.header}>
        <TouchableOpacity onPress={() => router.back()}>
          <Ionicons name="arrow-back" size={24} color={colors.gray900} />
        </TouchableOpacity>
        <View style={styles.headerInfo}>
          <Text style={styles.headerName}>{otherUser?.full_name}</Text>
          <Text style={styles.headerRole}>{otherUser?.role}</Text>
        </View>
        <TouchableOpacity onPress={() => {/* Show user info */}}>
          <Ionicons name="information-circle" size={24} color={colors.gray500} />
        </TouchableOpacity>
      </View>

      {/* Messages List */}
      <FlatList
        ref={flatListRef}
        data={messages}
        renderItem={({ item }) => (
          <MessageBubble
            message={item}
            isOwn={item.sender_id === user?.id}
          />
        )}
        keyExtractor={(item) => item.id}
        contentContainerStyle={styles.messagesList}
        onContentSizeChange={() => flatListRef.current?.scrollToEnd()}
      />

      {/* Input Area */}
      <KeyboardAvoidingView
        behavior={Platform.OS === 'ios' ? 'padding' : undefined}
        keyboardVerticalOffset={100}
      >
        <View style={styles.inputContainer}>
          <TextInput
            style={styles.textInput}
            value={newMessage}
            onChangeText={setNewMessage}
            placeholder="Type a message..."
            placeholderTextColor={colors.gray500}
            multiline
            maxLength={500}
          />
          <TouchableOpacity
            style={[
              styles.sendButton,
              (!newMessage.trim() || sending) && styles.sendButtonDisabled
            ]}
            onPress={sendMessage}
            disabled={!newMessage.trim() || sending}
          >
            {sending ? (
              <ActivityIndicator size="small" color={colors.white} />
            ) : (
              <Ionicons name="send" size={20} color={colors.white} />
            )}
          </TouchableOpacity>
        </View>
      </KeyboardAvoidingView>
    </SafeAreaView>
  );
}

// Styles for messages screen
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: colors.gray50,
  },
  tabContainer: {
    flexDirection: 'row',
    backgroundColor: colors.white,
    paddingHorizontal: spacing.md,
    paddingVertical: spacing.sm,
    borderBottomWidth: 1,
    borderBottomColor: colors.gray100,
  },
  tab: {
    flex: 1,
    paddingVertical: spacing.sm,
    alignItems: 'center',
    position: 'relative',
  },
  tabActive: {
    borderBottomWidth: 2,
    borderBottomColor: colors.primary,
  },
  tabText: {
    ...typography.body,
    color: colors.gray500,
  },
  tabTextActive: {
    color: colors.primary,
    fontFamily: 'Inter_600SemiBold',
  },
  badge: {
    position: 'absolute',
    top: 8,
    right: '30%',
    width: 8,
    height: 8,
    borderRadius: 4,
    backgroundColor: colors.error,
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  listContent: {
    paddingBottom: spacing.xl,
  },
  fab: {
    position: 'absolute',
    bottom: spacing.xl,
    right: spacing.lg,
    width: 56,
    height: 56,
    borderRadius: 28,
    backgroundColor: colors.primary,
    alignItems: 'center',
    justifyContent: 'center',
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.2,
    shadowRadius: 8,
    elevation: 5,
  },
});
```

#### 2. Message Components

```typescript
// components/messages/ConversationItem.tsx
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { Ionicons } from '@expo/vector-icons';
import { colors, spacing, typography } from '@/styles/theme';
import { formatRelativeTime } from '@/utils/formatters';

interface ConversationItemProps {
  otherUser: {
    id: string;
    full_name: string;
    role: string;
  };
  lastMessage: {
    message: string;
    created_at: string;
    sender_id: string;
  };
  unreadCount: number;
  onPress: () => void;
}

export const ConversationItem: React.FC<ConversationItemProps> = ({
  otherUser,
  lastMessage,
  unreadCount,
  onPress,
}) => {
  return (
    <TouchableOpacity style={styles.container} onPress={onPress}>
      <View style={styles.avatar}>
        <Ionicons name="person-circle" size={50} color={colors.gray400} />
      </View>
      
      <View style={styles.content}>
        <View style={styles.header}>
          <Text style={styles.name}>{otherUser.full_name}</Text>
          <Text style={styles.time}>
            {formatRelativeTime(lastMessage.created_at)}
          </Text>
        </View>
        
        <View style={styles.messageRow}>
          <Text style={styles.message} numberOfLines={1}>
            {lastMessage.message}
          </Text>
          {unreadCount > 0 && (
            <View style={styles.unreadBadge}>
              <Text style={styles.unreadCount}>{unreadCount}</Text>
            </View>
          )}
        </View>
      </View>
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    padding: spacing.md,
    backgroundColor: colors.white,
    borderBottomWidth: 1,
    borderBottomColor: colors.gray100,
  },
  avatar: {
    marginRight: spacing.md,
  },
  content: {
    flex: 1,
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: spacing.xs,
  },
  name: {
    ...typography.body,
    color: colors.gray900,
    fontFamily: 'Inter_600SemiBold',
  },
  time: {
    ...typography.caption,
    color: colors.gray500,
  },
  messageRow: {
    flexDirection: 'row',
    alignItems: 'center',
  },
  message: {
    ...typography.body,
    color: colors.gray600,
    flex: 1,
  },
  unreadBadge: {
    backgroundColor: colors.primary,
    borderRadius: 10,
    minWidth: 20,
    height: 20,
    alignItems: 'center',
    justifyContent: 'center',
    paddingHorizontal: spacing.xs,
  },
  unreadCount: {
    ...typography.caption,
    color: colors.white,
    fontFamily: 'Inter_600SemiBold',
    fontSize: 11,
  },
});

// components/messages/AnnouncementCard.tsx
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { Ionicons } from '@expo/vector-icons';
import { colors, spacing, typography } from '@/styles/theme';
import { formatRelativeTime } from '@/utils/formatters';

interface AnnouncementCardProps {
  title: string;
  content: string;
  author: string;
  createdAt: string;
  onPress: () => void;
}

export const AnnouncementCard: React.FC<AnnouncementCardProps> = ({
  title,
  content,
  author,
  createdAt,
  onPress,
}) => {
  return (
    <TouchableOpacity style={styles.container} onPress={onPress}>
      <View style={styles.iconContainer}>
        <Ionicons name="megaphone" size={24} color={colors.primary} />
      </View>
      
      <View style={styles.content}>
        <Text style={styles.title}>{title}</Text>
        <Text style={styles.message} numberOfLines={2}>
          {content}
        </Text>
        <View style={styles.footer}>
          <Text style={styles.author}>From {author}</Text>
          <Text style={styles.time}>{formatRelativeTime(createdAt)}</Text>
        </View>
      </View>
      
      <Ionicons name="chevron-forward" size={20} color={colors.gray400} />
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  container: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: colors.white,
    margin: spacing.sm,
    marginBottom: 0,
    padding: spacing.md,
    borderRadius: 12,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.05,
    shadowRadius: 4,
    elevation: 2,
  },
  iconContainer: {
    width: 48,
    height: 48,
    borderRadius: 24,
    backgroundColor: colors.primary + '10',
    alignItems: 'center',
    justifyContent: 'center',
    marginRight: spacing.md,
  },
  content: {
    flex: 1,
  },
  title: {
    ...typography.body,
    color: colors.gray900,
    fontFamily: 'Inter_600SemiBold',
    marginBottom: spacing.xs,
  },
  message: {
    ...typography.caption,
    color: colors.gray600,
    marginBottom: spacing.xs,
  },
  footer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
  },
  author: {
    ...typography.caption,
    color: colors.gray500,
  },
  time: {
    ...typography.caption,
    color: colors.gray500,
  },
});

// components/messages/MessageBubble.tsx
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { colors, spacing, typography } from '@/styles/theme';

interface MessageBubbleProps {
  message: {
    id: string;
    message: string;
    created_at: string;
    read: boolean;
  };
  isOwn: boolean;
}

export const MessageBubble: React.FC<MessageBubbleProps> = ({
  message,
  isOwn,
}) => {
  return (
    <View style={[
      styles.container,
      isOwn ? styles.ownContainer : styles.otherContainer
    ]}>
      <View style={[
        styles.bubble,
        isOwn ? styles.ownBubble : styles.otherBubble
      ]}>
        <Text style={[
          styles.text,
          isOwn ? styles.ownText : styles.otherText
        ]}>
          {message.message}
        </Text>
        <Text style={[
          styles.time,
          isOwn ? styles.ownTime : styles.otherTime
        ]}>
          {new Date(message.created_at).toLocaleTimeString([], {
            hour: '2-digit',
            minute: '2-digit'
          })}
        </Text>
      </View>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    paddingHorizontal: spacing.md,
    paddingVertical: spacing.xs,
  },
  ownContainer: {
    alignItems: 'flex-end',
  },
  otherContainer: {
    alignItems: 'flex-start',
  },
  bubble: {
    maxWidth: '80%',
    padding: spacing.sm,
    borderRadius: 16,
  },
  ownBubble: {
    backgroundColor: colors.primary,
    borderBottomRightRadius: 4,
  },
  otherBubble: {
    backgroundColor: colors.gray100,
    borderBottomLeftRadius: 4,
  },
  text: {
    ...typography.body,
  },
  ownText: {
    color: colors.white,
  },
  otherText: {
    color: colors.gray900,
  },
  time: {
    ...typography.caption,
    marginTop: spacing.xs,
  },
  ownTime: {
    color: colors.white + '90',
  },
  otherTime: {
    color: colors.gray500,
  },
});
```

#### 3. Chat Screen Styles

```typescript
// Styles for chat screen
const chatStyles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: colors.gray50,
  },
  loadingContainer: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
  },
  header: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: colors.white,
    paddingHorizontal: spacing.md,
    paddingVertical: spacing.sm,
    borderBottomWidth: 1,
    borderBottomColor: colors.gray100,
  },
  headerInfo: {
    flex: 1,
    marginLeft: spacing.md,
  },
  headerName: {
    ...typography.body,
    color: colors.gray900,
    fontFamily: 'Inter_600SemiBold',
  },
  headerRole: {
    ...typography.caption,
    color: colors.gray500,
  },
  messagesList: {
    paddingVertical: spacing.sm,
  },
  inputContainer: {
    flexDirection: 'row',
    backgroundColor: colors.white,
    borderTopWidth: 1,
    borderTopColor: colors.gray100,
    padding: spacing.md,
    alignItems: 'flex-end',
  },
  textInput: {
    flex: 1,
    backgroundColor: colors.gray50,
    borderRadius: 20,
    paddingHorizontal: spacing.md,
    paddingVertical: spacing.sm,
    marginRight: spacing.sm,
    maxHeight: 100,
    ...typography.body,
    color: colors.gray900,
  },
  sendButton: {
    width: 40,
    height: 40,
    borderRadius: 20,
    backgroundColor: colors.primary,
    alignItems: 'center',
    justifyContent: 'center',
  },
  sendButtonDisabled: {
    backgroundColor: colors.gray300,
  },
});
```

### Testing Requirements
- [ ] Message list loads correctly
- [ ] Real-time messaging works
- [ ] Read status updates
- [ ] Announcements display
- [ ] Push notifications trigger

### Acceptance Criteria
- Messages display in correct order
- Real-time updates work instantly
- Unread counts are accurate
- Tab switching is smooth
- Chat interface is responsive

---

## FRONTEND-008: Component Library & Design System

**Priority**: Medium  
**Estimated Hours**: 8  
**Dependencies**: FRONTEND-001

### Objective
Create all reusable UI components following the simplified design system.

### Component Implementations

#### 1. Button Component
```typescript
// components/ui/Button.tsx
import React from 'react';
import {
  TouchableOpacity,
  Text,
  ActivityIndicator,
  StyleSheet,
  ViewStyle,
  TextStyle,
} from 'react-native';
import { colors, spacing, typography } from '@/styles/theme';

interface ButtonProps {
  title: string;
  onPress: () => void;
  variant?: 'primary' | 'secondary' | 'ghost';
  size?: 'small' | 'medium' | 'large';
  loading?: boolean;
  disabled?: boolean;
  icon?: React.ReactNode;
  style?: ViewStyle;
  textStyle?: TextStyle;
}

export const Button: React.FC<ButtonProps> = ({
  title,
  onPress,
  variant = 'primary',
  size = 'medium',
  loading = false,
  disabled = false,
  icon,
  style,
  textStyle,
}) => {
  const isDisabled = disabled || loading;

  return (
    <TouchableOpacity
      style={[
        styles.base,
        styles[variant],
        styles[size],
        isDisabled && styles.disabled,
        style,
      ]}
      onPress={onPress}
      disabled={isDisabled}
      activeOpacity={0.7}
    >
      {loading ? (
        <ActivityIndicator
          size="small"
          color={variant === 'primary' ? colors.white : colors.primary}
        />
      ) : (
        <>
          {icon}
          <Text
            style={[
              styles.text,
              styles[`text_${variant}`],
              styles[`text_${size}`],
              textStyle,
            ]}
          >
            {title}
          </Text>
        </>
      )}
    </TouchableOpacity>
  );
};

const styles = StyleSheet.create({
  base: {
    flexDirection: 'row',
    alignItems: 'center',
    justifyContent: 'center',
    borderRadius: 8,
    gap: spacing.sm,
  },
  primary: {
    backgroundColor: colors.primary,
  },
  secondary: {
    backgroundColor: 'transparent',
    borderWidth: 1,
    borderColor: colors.primary,
  },
  ghost: {
    backgroundColor: 'transparent',
  },
  small: {
    paddingVertical: spacing.sm,
    paddingHorizontal: spacing.md,
  },
  medium: {
    paddingVertical: spacing.md,
    paddingHorizontal: spacing.lg,
  },
  large: {
    paddingVertical: spacing.md,
    paddingHorizontal: spacing.xl,
  },
  disabled: {
    opacity: 0.6,
  },
  text: {
    ...typography.body,
    fontFamily: 'Inter_600SemiBold',
  },
  text_primary: {
    color: colors.white,
  },
  text_secondary: {
    color: colors.primary,
  },
  text_ghost: {
    color: colors.gray700,
  },
  text_small: {
    fontSize: 14,
  },
  text_medium: {
    fontSize: 16,
  },
  text_large: {
    fontSize: 18,
  },
});
```

#### 2. Input Component
```typescript
// components/ui/Input.tsx
import React, { useState } from 'react';
import {
  View,
  Text,
  TextInput,
  TextInputProps,
  StyleSheet,
} from 'react-native';
import { Ionicons } from '@expo/vector-icons';
import { colors, spacing, typography } from '@/styles/theme';

interface InputProps extends TextInputProps {
  label?: string;
  error?: string;
  helper?: string;
  icon?: keyof typeof Ionicons.glyphMap;
  rightElement?: React.ReactNode;
}

export const Input: React.FC<InputProps> = ({
  label,
  error,
  helper,
  icon,
  rightElement,
  style,
  ...props
}) => {
  const [isFocused, setIsFocused] = useState(false);

  return (
    <View style={styles.container}>
      {label && <Text style={styles.label}>{label}</Text>}
      
      <View style={[
        styles.inputContainer,
        isFocused && styles.inputContainerFocused,
        error && styles.inputContainerError,
      ]}>
        {icon && (
          <Ionicons
            name={icon}
            size={20}
            color={error ? colors.error : isFocused ? colors.primary : colors.gray500}
            style={styles.icon}
          />
        )}
        
        <TextInput
          style={[styles.input, style]}
          placeholderTextColor={colors.gray500}
          onFocus={() => setIsFocused(true)}
          onBlur={() => setIsFocused(false)}
          {...props}
        />
        
        {rightElement}
      </View>
      
      {error && (
        <Text style={styles.error}>{error}</Text>
      )}
      
      {helper && !error && (
        <Text style={styles.helper}>{helper}</Text>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    marginBottom: spacing.md,
  },
  label: {
    ...typography.label,
    color: colors.gray700,
    marginBottom: spacing.xs,
    textTransform: 'uppercase',
  },
  inputContainer: {
    flexDirection: 'row',
    alignItems: 'center',
    borderWidth: 1,
    borderColor: colors.gray300,
    borderRadius: 8,
    backgroundColor: colors.white,
    paddingHorizontal: spacing.md,
  },
  inputContainerFocused: {
    borderColor: colors.primary,
    borderWidth: 2,
  },
  inputContainerError: {
    borderColor: colors.error,
  },
  icon: {
    marginRight: spacing.sm,
  },
  input: {
    flex: 1,
    paddingVertical: spacing.md,
    ...typography.body,
    color: colors.gray900,
  },
  error: {
    ...typography.caption,
    color: colors.error,
    marginTop: spacing.xs,
  },
  helper: {
    ...typography.caption,
    color: colors.gray500,
    marginTop: spacing.xs,
  },
});
```

#### 3. Card Component
```typescript
// components/ui/Card.tsx
import React from 'react';
import {
  View,
  TouchableOpacity,
  StyleSheet,
  ViewStyle,
} from 'react-native';
import { colors, spacing } from '@/styles/theme';

interface CardProps {
  children: React.ReactNode;
  onPress?: () => void;
  style?: ViewStyle;
  variant?: 'elevated' | 'outlined';
}

export const Card: React.FC<CardProps> = ({
  children,
  onPress,
  style,
  variant = 'elevated',
}) => {
  const Component = onPress ? TouchableOpacity : View;

  return (
    <Component
      style={[
        styles.base,
        variant === 'elevated' ? styles.elevated : styles.outlined,
        style,
      ]}
      onPress={onPress}
      activeOpacity={onPress ? 0.7 : 1}
    >
      {children}
    </Component>
  );
};

const styles = StyleSheet.create({
  base: {
    backgroundColor: colors.white,
    borderRadius: 12,
    padding: spacing.md,
    marginVertical: spacing.sm,
  },
  elevated: {
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
  },
  outlined: {
    borderWidth: 1,
    borderColor: colors.gray200,
  },
});
```

#### 4. Empty State Component
```typescript
// components/common/EmptyState.tsx
import React from 'react';
import { View, Text, TouchableOpacity, StyleSheet } from 'react-native';
import { Ionicons } from '@expo/vector-icons';
import { colors, spacing, typography } from '@/styles/theme';

interface EmptyStateProps {
  icon: keyof typeof Ionicons.glyphMap;
  title: string;
  description: string;
  action?: {
    label: string;
    onPress: () => void;
  };
}

export const EmptyState: React.FC<EmptyStateProps> = ({
  icon,
  title,
  description,
  action,
}) => {
  return (
    <View style={styles.container}>
      <View style={styles.iconContainer}>
        <Ionicons name={icon} size={64} color={colors.gray300} />
      </View>
      <Text style={styles.title}>{title}</Text>
      <Text style={styles.description}>{description}</Text>
      {action && (
        <TouchableOpacity style={styles.button} onPress={action.onPress}>
          <Text style={styles.buttonText}>{action.label}</Text>
        </TouchableOpacity>
      )}
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: spacing.xl,
    minHeight: 400,
  },
  iconContainer: {
    marginBottom: spacing.lg,
    opacity: 0.5,
  },
  title: {
    ...typography.h3,
    color: colors.gray900,
    marginBottom: spacing.sm,
    textAlign: 'center',
  },
  description: {
    ...typography.body,
    color: colors.gray500,
    textAlign: 'center',
    marginBottom: spacing.xl,
  },
  button: {
    backgroundColor: colors.primary,
    paddingVertical: spacing.sm,
    paddingHorizontal: spacing.lg,
    borderRadius: 8,
  },
  buttonText: {
    ...typography.body,
    color: colors.white,
    fontFamily: 'Inter_600SemiBold',
  },
});

// components/common/Toast.tsx
import React, { useEffect, useRef } from 'react';
import {
  Animated,
  Text,
  View,
  StyleSheet,
  TouchableOpacity,
} from 'react-native';
import { Ionicons } from '@expo/vector-icons';
import { colors, spacing, typography } from '@/styles/theme';

interface ToastProps {
  message: string;
  type: 'success' | 'error' | 'info' | 'warning';
  duration?: number;
  onClose?: () => void;
}

export const Toast: React.FC<ToastProps> = ({
  message,
  type,
  duration = 3000,
  onClose,
}) => {
  const opacity = useRef(new Animated.Value(0)).current;
  const translateY = useRef(new Animated.Value(-100)).current;

  useEffect(() => {
    // Animate in
    Animated.parallel([
      Animated.timing(opacity, {
        toValue: 1,
        duration: 300,
        useNativeDriver: true,
      }),
      Animated.timing(translateY, {
        toValue: 0,
        duration: 300,
        useNativeDriver: true,
      }),
    ]).start();

    // Auto dismiss
    const timer = setTimeout(() => {
      dismiss();
    }, duration);

    return () => clearTimeout(timer);
  }, []);

  const dismiss = () => {
    Animated.parallel([
      Animated.timing(opacity, {
        toValue: 0,
        duration: 300,
        useNativeDriver: true,
      }),
      Animated.timing(translateY, {
        toValue: -100,
        duration: 300,
        useNativeDriver: true,
      }),
    ]).start(() => {
      onClose?.();
    });
  };

  const getIcon = () => {
    switch (type) {
      case 'success':
        return 'checkmark-circle';
      case 'error':
        return 'close-circle';
      case 'warning':
        return 'warning';
      default:
        return 'information-circle';
    }
  };

  const getColor = () => {
    switch (type) {
      case 'success':
        return colors.success;
      case 'error':
        return colors.error;
      case 'warning':
        return colors.warning;
      default:
        return colors.info;
    }
  };

  return (
    <Animated.View
      style={[
        styles.container,
        {
          opacity,
          transform: [{ translateY }],
        },
      ]}
    >
      <TouchableOpacity
        style={[styles.toast, { borderLeftColor: getColor() }]}
        onPress={dismiss}
        activeOpacity={0.9}
      >
        <Ionicons name={getIcon()} size={24} color={getColor()} />
        <Text style={styles.message}>{message}</Text>
        <Ionicons name="close" size={20} color={colors.gray500} />
      </TouchableOpacity>
    </Animated.View>
  );
};

const styles = StyleSheet.create({
  container: {
    position: 'absolute',
    top: 50,
    left: spacing.md,
    right: spacing.md,
    zIndex: 9999,
  },
  toast: {
    backgroundColor: colors.white,
    borderRadius: 8,
    borderLeftWidth: 4,
    flexDirection: 'row',
    alignItems: 'center',
    padding: spacing.md,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 4 },
    shadowOpacity: 0.1,
    shadowRadius: 8,
    elevation: 5,
    gap: spacing.sm,
  },
  message: {
    ...typography.body,
    color: colors.gray900,
    flex: 1,
  },
});

// components/common/LoadingSpinner.tsx
import React from 'react';
import { View, ActivityIndicator, StyleSheet } from 'react-native';
import { colors } from '@/styles/theme';

interface LoadingSpinnerProps {
  size?: 'small' | 'large';
  color?: string;
  fullScreen?: boolean;
}

export const LoadingSpinner: React.FC<LoadingSpinnerProps> = ({
  size = 'large',
  color = colors.primary,
  fullScreen = false,
}) => {
  if (fullScreen) {
    return (
      <View style={styles.fullScreen}>
        <ActivityIndicator size={size} color={color} />
      </View>
    );
  }

  return <ActivityIndicator size={size} color={color} />;
};

const styles = StyleSheet.create({
  fullScreen: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: colors.white,
  },
});
```

### Testing Requirements
- [ ] All components render correctly
- [ ] Props work as expected
- [ ] Styling is consistent
- [ ] Accessibility features work
- [ ] Dark mode support (if implemented)

### Acceptance Criteria
- Components are reusable across screens
- Consistent styling throughout app
- Props provide proper customization
- TypeScript types are complete
- Performance is optimized

---

## FRONTEND-009: State Management & Data Synchronization

**Priority**: High  
**Estimated Hours**: 10  
**Dependencies**: FRONTEND-008

### Objective
Implement Zustand stores for state management and Supabase real-time synchronization.

### Implementation Details

#### 1. Auth Store
```typescript
// stores/authStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { supabase } from '@/lib/supabase';

interface Profile {
  id: string;
  full_name: string;
  phone_number: string;
  office_id: string;
  upline_id?: string;
  role: 'member' | 'leader' | 'admin';
  office?: any;
}

interface AuthState {
  user: Profile | null;
  session: any | null;
  isLoading: boolean;
  setUser: (user: Profile | null) => void;
  setSession: (session: any | null) => void;
  initialize: () => Promise<void>;
  signOut: () => Promise<void>;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set, get) => ({
      user: null,
      session: null,
      isLoading: true,

      setUser: (user) => set({ user }),
      setSession: (session) => set({ session }),

      initialize: async () => {
        try {
          const { data: { session } } = await supabase.auth.getSession();
          
          if (session?.user) {
            const { data: profile } = await supabase
              .from('profiles')
              .select('*, office:offices(*)')
              .eq('id', session.user.id)
              .single();

            set({ 
              session, 
              user: profile, 
              isLoading: false 
            });
          } else {
            set({ 
              session: null, 
              user: null, 
              isLoading: false 
            });
          }
        } catch (error) {
          console.error('Auth initialization error:', error);
          set({ isLoading: false });
        }
      },

      signOut: async () => {
        try {
          await supabase.auth.signOut();
          set({ user: null, session: null });
        } catch (error) {
          console.error('Sign out error:', error);
        }
      },
    }),
    {
      name: 'auth-storage',
      storage: createJSONStorage(() => AsyncStorage),
      partialize: (state) => ({ 
        user: state.user,
        session: state.session,
      }),
    }
  )
);
```

#### 2. Content Store
```typescript
// stores/contentStore.ts
import { create } from 'zustand';
import { supabase } from '@/lib/supabase';

interface ContentIdea {
  id: string;
  title: string;
  description: string;
  leader_comment: string;
  theme_week: number;
  created_at: string;
  leader: any;
}

interface ContentState {
  contentIdeas: ContentIdea[];
  loading: boolean;
  filter: 'all' | 'pending' | 'completed';
  setContentIdeas: (ideas: ContentIdea[]) => void;
  setFilter: (filter: 'all' | 'pending' | 'completed') => void;
  fetchContent: (userId: string) => Promise<void>;
  markAsCompleted: (ideaId: string) => void;
}

export const useContentStore = create<ContentState>((set, get) => ({
  contentIdeas: [],
  loading: false,
  filter: 'all',

  setContentIdeas: (contentIdeas) => set({ contentIdeas }),
  setFilter: (filter) => set({ filter }),

  fetchContent: async (userId) => {
    set({ loading: true });
    try {
      const { data } = await supabase
        .from('content_ideas')
        .select(`
          *,
          leader:leader_id(full_name, role),
          submission:content_submissions!left(passed, created_at)
        `)
        .eq('active', true)
        .order('created_at', { ascending: false });

      set({ contentIdeas: data || [], loading: false });
    } catch (error) {
      console.error('Error fetching content:', error);
      set({ loading: false });
    }
  },

  markAsCompleted: (ideaId) => {
    const { contentIdeas } = get();
    const updated = contentIdeas.map(idea => 
      idea.id === ideaId 
        ? { ...idea, submission: { passed: true, created_at: new Date().toISOString() } }
        : idea
    );
    set({ contentIdeas: updated });
  },
}));
```

#### 3. Analytics Store
```typescript
// stores/analyticsStore.ts
import { create } from 'zustand';
import { supabase } from '@/lib/supabase';

interface AnalyticsState {
  todayClicks: number;
  weekClicks: number;
  totalClicks: number;
  topPost: any | null;
  recentLeads: any[];
  loading: boolean;
  fetchAnalytics: (userId: string) => Promise<void>;
  incrementClicks: () => void;
  addLead: (lead: any) => void;
}

export const useAnalyticsStore = create<AnalyticsState>((set, get) => ({
  todayClicks: 0,
  weekClicks: 0,
  totalClicks: 0,
  topPost: null,
  recentLeads: [],
  loading: false,

  fetchAnalytics: async (userId) => {
    set({ loading: true });
    try {
      // Fetch analytics data
      const today = new Date().toISOString().split('T')[0];
      const weekAgo = new Date();
      weekAgo.setDate(weekAgo.getDate() - 7);

      const { data: analytics } = await supabase
        .from('link_analytics')
        .select('*')
        .eq('user_id', userId)
        .gte('last_clicked', weekAgo.toISOString());

      // Calculate metrics
      let todayClicks = 0;
      let weekClicks = 0;
      let totalClicks = 0;

      analytics?.forEach(item => {
        totalClicks += item.total_clicks;
        if (item.last_clicked?.startsWith(today)) {
          todayClicks += item.total_clicks;
        }
        weekClicks += item.total_clicks;
      });

      // Get top post
      const topPost = analytics?.sort((a, b) => b.total_clicks - a.total_clicks)[0];

      set({
        todayClicks,
        weekClicks,
        totalClicks,
        topPost,
        loading: false,
      });
    } catch (error) {
      console.error('Error fetching analytics:', error);
      set({ loading: false });
    }
  },

  incrementClicks: () => {
    set(state => ({
      todayClicks: state.todayClicks + 1,
      weekClicks: state.weekClicks + 1,
      totalClicks: state.totalClicks + 1,
    }));
  },

  addLead: (lead) => {
    set(state => ({
      recentLeads: [lead, ...state.recentLeads].slice(0, 10),
    }));
  },
}));
```

#### 4. Notification Store
```typescript
// stores/notificationStore.ts
import { create } from 'zustand';
import * as Notifications from 'expo-notifications';
import { supabase } from '@/lib/supabase';

interface NotificationState {
  pushToken: string | null;
  notifications: Notification[];
  unreadCount: number;
  setPushToken: (token: string) => void;
  addNotification: (notification: Notification) => void;
  markAsRead: (id: string) => void;
  clearAll: () => void;
  registerForPushNotifications: () => Promise<void>;
}

interface Notification {
  id: string;
  title: string;
  body: string;
  data?: any;
  createdAt: Date;
  read: boolean;
}

export const useNotificationStore = create<NotificationState>((set, get) => ({
  pushToken: null,
  notifications: [],
  unreadCount: 0,

  setPushToken: (pushToken) => set({ pushToken }),

  addNotification: (notification) => {
    set(state => ({
      notifications: [notification, ...state.notifications],
      unreadCount: state.unreadCount + 1,
    }));
  },

  markAsRead: (id) => {
    set(state => {
      const notifications = state.notifications.map(n =>
        n.id === id ? { ...n, read: true } : n
      );
      const unreadCount = notifications.filter(n => !n.read).length;
      return { notifications, unreadCount };
    });
  },

  clearAll: () => set({ notifications: [], unreadCount: 0 }),

  registerForPushNotifications: async () => {
    try {
      const { status: existingStatus } = await Notifications.getPermissionsAsync();
      let finalStatus = existingStatus;
      
      if (existingStatus !== 'granted') {
        const { status } = await Notifications.requestPermissionsAsync();
        finalStatus = status;
      }
      
      if (finalStatus !== 'granted') {
        console.log('Failed to get push token for push notification!');
        return;
      }
      
      const token = (await Notifications.getExpoPushTokenAsync()).data;
      set({ pushToken: token });
      
      // Save token to user profile
      const { data: { user } } = await supabase.auth.getUser();
      if (user) {
        await supabase
          .from('profiles')
          .update({ push_token: token })
          .eq('id', user.id);
      }
    } catch (error) {
      console.error('Error registering for push notifications:', error);
    }
  },
}));
```

#### 5. Real-time Subscription Manager
```typescript
// lib/realtimeManager.ts
import { supabase } from './supabase';
import { useAuthStore } from '@/stores/authStore';
import { useContentStore } from '@/stores/contentStore';
import { useAnalyticsStore } from '@/stores/analyticsStore';
import { useNotificationStore } from '@/stores/notificationStore';

class RealtimeManager {
  private subscriptions: Map<string, any> = new Map();

  initialize(userId: string) {
    this.setupContentSubscription(userId);
    this.setupAnalyticsSubscription(userId);
    this.setupMessageSubscription(userId);
    this.setupLeadSubscription(userId);
  }

  private setupContentSubscription(userId: string) {
    const subscription = supabase
      .channel('content_changes')
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          table: 'content_ideas',
        },
        (payload) => {
          const contentStore = useContentStore.getState();
          contentStore.fetchContent(userId);
          
          // Show notification
          const notificationStore = useNotificationStore.getState();
          notificationStore.addNotification({
            id: Date.now().toString(),
            title: 'New Content Idea!',
            body: payload.new.title,
            createdAt: new Date(),
            read: false,
          });
        }
      )
      .subscribe();

    this.subscriptions.set('content', subscription);
  }

  private setupAnalyticsSubscription(userId: string) {
    const subscription = supabase
      .channel('analytics_changes')
      .on(
        'postgres_changes',
        {
          event: 'UPDATE',
          schema: 'public',
          table: 'link_analytics',
          filter: `user_id=eq.${userId}`,
        },
        () => {
          const analyticsStore = useAnalyticsStore.getState();
          analyticsStore.incrementClicks();
        }
      )
      .subscribe();

    this.subscriptions.set('analytics', subscription);
  }

  private setupMessageSubscription(userId: string) {
    const subscription = supabase
      .channel('messages_channel')
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          table: 'messages',
          filter: `recipient_id=eq.${userId}`,
        },
        (payload) => {
          // Show notification
          const notificationStore = useNotificationStore.getState();
          notificationStore.addNotification({
            id: payload.new.id,
            title: 'New Message',
            body: payload.new.message,
            data: { type: 'message', senderId: payload.new.sender_id },
            createdAt: new Date(),
            read: false,
          });
        }
      )
      .subscribe();

    this.subscriptions.set('messages', subscription);
  }

  private setupLeadSubscription(userId: string) {
    const subscription = supabase
      .channel('leads_channel')
      .on(
        'postgres_changes',
        {
          event: 'INSERT',
          schema: 'public',
          table: 'lead_notifications',
          filter: `user_id=eq.${userId}`,
        },
        (payload) => {
          const analyticsStore = useAnalyticsStore.getState();
          analyticsStore.addLead(payload.new);
          
          // Show notification
          const notificationStore = useNotificationStore.getState();
          notificationStore.addNotification({
            id: payload.new.id,
            title: 'New Lead! 🎉',
            body: 'Someone clicked your link just now',
            data: { type: 'lead', trackingId: payload.new.tracking_id },
            createdAt: new Date(),
            read: false,
          });
        }
      )
      .subscribe();

    this.subscriptions.set('leads', subscription);
  }

  cleanup() {
    this.subscriptions.forEach(subscription => {
      subscription.unsubscribe();
    });
    this.subscriptions.clear();
  }
}

export const realtimeManager = new RealtimeManager();
```

### Testing Requirements
- [ ] State persists across app restarts
- [ ] Real-time updates sync to store
- [ ] Store actions work correctly
- [ ] No memory leaks
- [ ] Offline support works

### Acceptance Criteria
- State management is centralized and efficient
- Real-time synchronization works seamlessly
- Offline changes sync when online
- No data loss on app restart
- Performance remains optimal

---

## FRONTEND-010: Performance Optimization & Polish

**Priority**: High  
**Estimated Hours**: 12  
**Dependencies**: All previous tickets

### Objective
Optimize app performance, implement error handling, add final polish, and prepare for production.

### Implementation Details

#### 1. Performance Optimizations
```typescript
// utils/performance.ts
import React from 'react';
import { Image } from 'react-native';
import FastImage from 'react-native-fast-image';

// Memoized components
export const MemoizedContentCard = React.memo(ContentCard);
export const MemoizedMessageBubble = React.memo(MessageBubble);
export const MemoizedChecklistItem = React.memo(ChecklistItem);

// Image optimization
export const OptimizedImage = FastImage;

// List optimization
export const listOptimizationProps = {
  removeClippedSubviews: true,
  maxToRenderPerBatch: 10,
  updateCellsBatchingPeriod: 50,
  initialNumToRender: 10,
  windowSize: 10,
  getItemLayout: (data: any, index: number) => ({
    length: ITEM_HEIGHT,
    offset: ITEM_HEIGHT * index,
    index,
  }),
};
```

#### 2. Error Boundaries
```typescript
// components/common/ErrorBoundary.tsx
import React, { Component, ErrorInfo } from 'react';
import { View, Text, TouchableOpacity } from 'react-native';
import { colors, spacing, typography } from '@/styles/theme';

interface Props {
  children: React.ReactNode;
  fallback?: React.ReactNode;
}

interface State {
  hasError: boolean;
  error: Error | null;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    // Send to error tracking service
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <View style={styles.container}>
          <Text style={styles.title}>Oops! Something went wrong</Text>
          <Text style={styles.message}>
            We're sorry for the inconvenience. Please try again.
          </Text>
          <TouchableOpacity
            style={styles.button}
            onPress={() => this.setState({ hasError: false, error: null })}
          >
            <Text style={styles.buttonText}>Try Again</Text>
          </TouchableOpacity>
        </View>
      );
    }

    return this.props.children;
  }
}

const styles = {
  // Error boundary styles
};
```

#### 3. App Configuration
```json
// app.json updates for production
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
    "updates": {
      "fallbackToCacheTimeout": 0,
      "url": "https://u.expo.dev/[project-id]"
    },
    "assetBundlePatterns": ["**/*"],
    "ios": {
      "supportsTablet": false,
      "bundleIdentifier": "com.galaxyteam.app",
      "buildNumber": "1",
      "infoPlist": {
        "NSCameraUsageDescription": "This app uses the camera to upload profile pictures.",
        "NSPhotoLibraryUsageDescription": "This app accesses your photos to upload profile pictures."
      }
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#0B5CFF"
      },
      "package": "com.galaxyteam.app",
      "versionCode": 1,
      "permissions": [
        "NOTIFICATIONS",
        "READ_EXTERNAL_STORAGE",
        "WRITE_EXTERNAL_STORAGE",
        "CAMERA"
      ]
    },
    "extra": {
      "eas": {
        "projectId": "[your-project-id]"
      }
    },
    "plugins": [
      "expo-router",
      [
        "expo-notifications",
        {
          "icon": "./assets/notification-icon.png",
          "color": "#0B5CFF",
          "sounds": ["./assets/notification-sound.wav"]
        }
      ]
    ]
  }
}
```

#### 4. Final Testing Checklist
```typescript
// __tests__/app.test.ts
describe('Galaxy Team App', () => {
  // Authentication Tests
  test('User can register with phone number', async () => {});
  test('OTP verification works', async () => {});
  test('Session persists after app restart', async () => {});

  // Content Tests
  test('Content ideas load correctly', async () => {});
  test('Content grading validates all rules', async () => {});
  test('Tracking ID generates correctly', async () => {});

  // Analytics Tests
  test('Click counts update in real-time', async () => {});
  test('Lead notifications trigger', async () => {});

  // Performance Tests
  test('App loads in under 3 seconds', async () => {});
  test('Lists scroll smoothly with 1000 items', async () => {});
  test('Memory usage stays under limit', async () => {});
});
```

#### 5. Splash Screen Configuration
```typescript
// app/+not-found.tsx
import { Link, Stack } from 'expo-router';
import { View, Text } from 'react-native';
import { colors, spacing, typography } from '@/styles/theme';

export default function NotFoundScreen() {
  return (
    <>
      <Stack.Screen options={{ title: 'Oops!' }} />
      <View style={styles.container}>
        <Text style={styles.title}>This screen doesn't exist.</Text>
        <Link href="/" style={styles.link}>
          <Text style={styles.linkText}>Go to home screen!</Text>
        </Link>
      </View>
    </>
  );
}

const styles = {
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
    padding: spacing.lg,
  },
  title: {
    ...typography.h2,
    color: colors.gray900,
    marginBottom: spacing.lg,
  },
  link: {
    marginTop: spacing.md,
    paddingVertical: spacing.md,
  },
  linkText: {
    ...typography.body,
    color: colors.primary,
  },
};

// utils/performanceMonitor.ts
import * as Updates from 'expo-updates';
import * as Device from 'expo-device';
import * as Application from 'expo-application';
import NetInfo from '@react-native-community/netinfo';

export class PerformanceMonitor {
  private metrics: Map<string, number> = new Map();
  private renderTimes: number[] = [];

  startMeasure(label: string) {
    this.metrics.set(label, Date.now());
  }

  endMeasure(label: string): number {
    const startTime = this.metrics.get(label);
    if (!startTime) return 0;
    
    const duration = Date.now() - startTime;
    this.metrics.delete(label);
    
    console.log(`[Performance] ${label}: ${duration}ms`);
    return duration;
  }

  trackRenderTime(componentName: string, renderTime: number) {
    this.renderTimes.push(renderTime);
    
    if (renderTime > 16) { // More than 1 frame at 60fps
      console.warn(`[Performance] Slow render in ${componentName}: ${renderTime}ms`);
    }
  }

  async getDeviceInfo() {
    const netInfo = await NetInfo.fetch();
    
    return {
      deviceName: Device.deviceName,
      deviceModel: Device.modelName,
      osVersion: Device.osVersion,
      appVersion: Application.nativeApplicationVersion,
      buildVersion: Application.nativeBuildVersion,
      isDevice: Device.isDevice,
      networkType: netInfo.type,
      isConnected: netInfo.isConnected,
      memoryUsage: (performance as any).memory?.usedJSHeapSize,
    };
  }

  getAverageRenderTime(): number {
    if (this.renderTimes.length === 0) return 0;
    
    const sum = this.renderTimes.reduce((a, b) => a + b, 0);
    return sum / this.renderTimes.length;
  }

  reset() {
    this.metrics.clear();
    this.renderTimes = [];
  }
}

export const performanceMonitor = new PerformanceMonitor();
```

#### 6. Production Build Configuration
```json
// eas.json
{
  "cli": {
    "version": ">= 5.0.0",
    "promptToConfigurePushNotifications": false
  },
  "build": {
    "development": {
      "developmentClient": true,
      "distribution": "internal",
      "android": {
        "buildType": "apk"
      },
      "ios": {
        "simulator": true
      }
    },
    "preview": {
      "distribution": "internal",
      "android": {
        "buildType": "apk"
      },
      "ios": {
        "simulator": false,
        "enterpriseProvisioning": "adhoc"
      },
      "env": {
        "EXPO_PUBLIC_SUPABASE_URL": "https://your-project.supabase.co",
        "EXPO_PUBLIC_SUPABASE_ANON_KEY": "your-anon-key"
      }
    },
    "production": {
      "android": {
        "buildType": "app-bundle",
        "gradle": {
          "extraPropertiesFile": "./android.properties"
        }
      },
      "ios": {
        "enterpriseProvisioning": "universal",
        "autoIncrement": true
      },
      "env": {
        "EXPO_PUBLIC_SUPABASE_URL": "https://your-project.supabase.co",
        "EXPO_PUBLIC_SUPABASE_ANON_KEY": "your-anon-key",
        "EXPO_PUBLIC_ERP_WEBHOOK_URL": "https://your-erp-domain.com/webhook"
      }
    }
  },
  "submit": {
    "production": {
      "android": {
        "serviceAccountKeyPath": "./google-service-account.json",
        "track": "production",
        "releaseStatus": "completed",
        "rollout": 0.1
      },
      "ios": {
        "appleId": "your-apple-id@email.com",
        "ascAppId": "1234567890",
        "appleTeamId": "TEAM_ID"
      }
    }
  }
}

// app.config.ts
import { ExpoConfig, ConfigContext } from 'expo/config';

export default ({ config }: ConfigContext): ExpoConfig => ({
  ...config,
  name: 'Galaxy Team',
  slug: 'galaxy-team',
  version: '1.0.0',
  orientation: 'portrait',
  icon: './assets/icon.png',
  userInterfaceStyle: 'automatic',
  splash: {
    image: './assets/splash.png',
    resizeMode: 'contain',
    backgroundColor: '#0B5CFF',
  },
  updates: {
    fallbackToCacheTimeout: 0,
    url: 'https://u.expo.dev/your-project-id',
  },
  assetBundlePatterns: ['**/*'],
  ios: {
    supportsTablet: false,
    bundleIdentifier: 'com.galaxyteam.app',
    config: {
      usesNonExemptEncryption: false,
    },
    infoPlist: {
      NSCameraUsageDescription: 'This app uses the camera to scan QR codes.',
      NSPhotoLibraryUsageDescription: 'This app accesses your photos for profile pictures.',
    },
  },
  android: {
    adaptiveIcon: {
      foregroundImage: './assets/adaptive-icon.png',
      backgroundColor: '#0B5CFF',
    },
    package: 'com.galaxyteam.app',
    permissions: [
      'NOTIFICATIONS',
      'READ_EXTERNAL_STORAGE',
      'WRITE_EXTERNAL_STORAGE',
    ],
    googleServicesFile: './google-services.json',
  },
  web: {
    favicon: './assets/favicon.png',
    bundler: 'metro',
  },
  plugins: [
    'expo-router',
    [
      'expo-notifications',
      {
        icon: './assets/notification-icon.png',
        color: '#0B5CFF',
        sounds: ['./assets/sounds/notification.wav'],
      },
    ],
    [
      'expo-build-properties',
      {
        android: {
          compileSdkVersion: 33,
          targetSdkVersion: 33,
          buildToolsVersion: '33.0.0',
        },
        ios: {
          deploymentTarget: '13.0',
        },
      },
    ],
  ],
  extra: {
    eas: {
      projectId: 'your-eas-project-id',
    },
    supabaseUrl: process.env.EXPO_PUBLIC_SUPABASE_URL,
    supabaseAnonKey: process.env.EXPO_PUBLIC_SUPABASE_ANON_KEY,
  },
});
```

#### 7. Utility Functions
```typescript
// utils/formatters.ts
export function formatPhoneNumber(phone: string): string {
  // Remove all non-digits
  const cleaned = phone.replace(/\D/g, '');
  
  // Add +251 if not present
  if (cleaned.startsWith('251')) {
    return `+${cleaned}`;
  } else if (cleaned.startsWith('0')) {
    return `+251${cleaned.substring(1)}`;
  } else {
    return `+251${cleaned}`;
  }
}

export function formatNumber(num: number): string {
  if (num >= 1000000) {
    return `${(num / 1000000).toFixed(1)}M`;
  } else if (num >= 1000) {
    return `${(num / 1000).toFixed(1)}K`;
  }
  return num.toString();
}

export function formatRelativeTime(dateString: string): string {
  const date = new Date(dateString);
  const now = new Date();
  const diffInSeconds = Math.floor((now.getTime() - date.getTime()) / 1000);

  if (diffInSeconds < 60) return 'just now';
  if (diffInSeconds < 3600) return `${Math.floor(diffInSeconds / 60)}m ago`;
  if (diffInSeconds < 86400) return `${Math.floor(diffInSeconds / 3600)}h ago`;
  if (diffInSeconds < 604800) return `${Math.floor(diffInSeconds / 86400)}d ago`;
  
  return date.toLocaleDateString();
}

// utils/storage.ts
import AsyncStorage from '@react-native-async-storage/async-storage';

export const storage = {
  async get(key: string): Promise<any> {
    try {
      const value = await AsyncStorage.getItem(key);
      return value ? JSON.parse(value) : null;
    } catch (error) {
      console.error('Storage get error:', error);
      return null;
    }
  },

  async set(key: string, value: any): Promise<void> {
    try {
      await AsyncStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('Storage set error:', error);
    }
  },

  async remove(key: string): Promise<void> {
    try {
      await AsyncStorage.removeItem(key);
    } catch (error) {
      console.error('Storage remove error:', error);
    }
  },

  async clear(): Promise<void> {
    try {
      await AsyncStorage.clear();
    } catch (error) {
      console.error('Storage clear error:', error);
    }
  },
};
```

### Testing Requirements
- [ ] All features work as expected
- [ ] No crashes or errors
- [ ] Performance is smooth
- [ ] Offline mode works
- [ ] Production build succeeds

### Acceptance Criteria
- App loads in under 3 seconds
- Lists scroll smoothly with 1000+ items
- Memory usage stays under 200MB
- No memory leaks detected
- All production optimizations applied
- Error tracking configured
- Analytics implemented
- App store ready

### Final Checklist
- [ ] All TypeScript errors resolved
- [ ] ESLint warnings addressed
- [ ] Assets optimized (images compressed)
- [ ] Splash screen configured
- [ ] App icons created for all sizes
- [ ] Environment variables secured
- [ ] Push notifications tested
- [ ] Deep linking configured
- [ ] App permissions documented
- [ ] Privacy policy updated
- [ ] Terms of service ready
- [ ] App store descriptions written
- [ ] Screenshots prepared
- [ ] Release notes drafted

## Project Completion Summary

These 10 comprehensive frontend tickets provide everything needed to build the Galaxy Team Internal App from scratch. The implementation includes:

### Technical Stack
- **Expo SDK 49+** for cross-platform development
- **TypeScript** for type safety
- **Supabase** for backend services
- **Zustand** for state management
- **React Hook Form** for form handling
- **Expo Router** for navigation

### Key Features Delivered
1. **Authentication System** - Phone-based OTP login
2. **Content Distribution** - Leader to member content flow
3. **Simple Grading** - 4-point validation system
4. **Analytics Dashboard** - Click tracking and leads
5. **Messaging System** - Upline communication
6. **Real-time Updates** - Live data synchronization
7. **Offline Support** - Works without internet
8. **Push Notifications** - Instant alerts

### Development Timeline
- **Week 1**: Project setup and authentication
- **Week 2-3**: Core features (content, grading, analytics)
- **Week 4**: Communication and messaging
- **Week 5**: Polish and optimization
- **Week 6**: Testing and deployment

### Deliverables
- Complete source code with TypeScript
- Reusable component library
- State management system
- Performance optimizations
- Production-ready configuration
- Comprehensive documentation

The app is designed to scale from 3,000 to 10,000+ users without significant changes, focusing on simplicity and efficiency throughout.