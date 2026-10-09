---
name: flutter-mobile
description: >-
  Use this skill when working on the Flutter 3 mobile app (apps/mobile/).
  Covers project structure, GoRouter navigation, Riverpod state management,
  Dio HTTP client setup, Razorpay Flutter SDK, FCM push notifications,
  and release build commands for Android and iOS.
---

# Flutter Mobile App Skill

## Project Location
All mobile app code lives in `apps/mobile/`.

## Key Commands
```bash
# Get dependencies
cd apps/mobile && flutter pub get

# Run on connected device/emulator
cd apps/mobile && flutter run

# Analyze code
cd apps/mobile && flutter analyze

# Run tests
cd apps/mobile && flutter test

# Build Android APK (debug)
cd apps/mobile && flutter build apk --debug

# Build Android APK (release)
cd apps/mobile && flutter build apk --release

# Build iOS (release) — macOS only
cd apps/mobile && flutter build ipa --release
```

## Architecture Pattern (Feature-First)
```
lib/
├── main.dart
├── core/
│   ├── api/          ← Dio HTTP client + interceptors
│   ├── auth/         ← OTP auth state
│   ├── router/       ← GoRouter route definitions
│   └── theme/        ← AppTheme, colours, typography
└── features/
    └── <feature>/
        ├── data/     ← Repository + API data sources
        ├── domain/   ← Models + use cases
        └── presentation/
            ├── pages/
            └── widgets/
```

## HTTP Client (Dio) Pattern
```dart
// core/api/api_client.dart
final dio = Dio(BaseOptions(baseUrl: Env.apiUrl));
dio.interceptors.add(AuthInterceptor()); // Attaches JWT Bearer token
```

## State Management (Riverpod)
```dart
// Use @riverpod code generation
@riverpod
Future<List<Listing>> searchListings(SearchListingsRef ref, SearchParams params) {
  return ref.watch(listingRepositoryProvider).search(params);
}
```

## OTP Login Flow
1. User enters +91 phone number
2. Call `POST /api/v1/auth/send-otp`
3. User enters 6-digit OTP
4. Call `POST /api/v1/auth/verify-otp` → receive `access_token` + `refresh_token`
5. Store tokens in `flutter_secure_storage`
6. Navigate to home

## Razorpay Payment
```dart
// Use razorpay_flutter package
final razorpay = Razorpay();
razorpay.on(Razorpay.EVENT_PAYMENT_SUCCESS, _handlePaymentSuccess);
razorpay.open({
  'key': Env.razorpayKeyId,
  'amount': booking.totalPaise,
  'currency': 'INR',
  'order_id': razorpayOrderId,
  'prefill': {'contact': user.phone},
});
```

## Key Packages
```yaml
# pubspec.yaml essentials
dependencies:
  flutter_riverpod: ^2.5.1
  riverpod_annotation: ^2.3.5
  go_router: ^14.0.0
  dio: ^5.4.3
  flutter_secure_storage: ^9.0.0
  razorpay_flutter: ^1.3.6
  firebase_messaging: ^15.0.0
  google_maps_flutter: ^2.9.0
  cached_network_image: ^3.3.1
  intl: ^0.19.0
```

## Environment Variables
Use `--dart-define-from-file=.env.json` at build time:
```json
{
  "API_URL": "https://api.emptyshopfinder.in",
  "RAZORPAY_KEY_ID": "rzp_test_...",
  "GOOGLE_MAPS_KEY": "..."
}
```
