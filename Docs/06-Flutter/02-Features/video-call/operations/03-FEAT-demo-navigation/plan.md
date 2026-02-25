---
doc_role: operation
operation_id: FEAT-demo-navigation
type: FEAT
status: in_progress
created_at: 2026-02-25
backend_reference:
  module: N/A
  operations: []
  path: N/A
---

# FEAT-demo-navigation — Flutter Plan

## 1. Objective

Provide a temporary navigation mechanism to access the demo video call screen (`/demo-video-call`) implemented in `FEAT-integrate-video-call-demo`. This navigation will be added to the bottom of the Member, Rescuer, and Expert home screens to allow easy testing of the video call functionality across different app roles.

## 2. As-Is (Flutter State)

- The `DemoVideoCallScreen` is fully implemented and mapped to the `/demo-video-call` route.
- The `go_router` package is used for navigation.
- `MemberHomeScreen` (`lib/features/member/screens/home_screen.dart`) has a Developer Tools section.
- `RescuerHomeScreen` (`lib/features/rescuer/screens/rescuer_home_screen.dart`) and `ExpertHomeScreen` (`lib/features/expert/screens/expert_home_screen.dart`) do not have access to the demo screen.

## 3. Gap Analysis

| Gap                          | Required Action                                                                                | Target Location                                                                                                                                                          |
| ---------------------------- | ---------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Missing Navigation Paths** | Add a "Video Call Demonstration" section with a button to navigate to the demo video call page | - `lib/features/member/screens/home_screen.dart`<br>- `lib/features/rescuer/screens/rescuer_home_screen.dart`<br>- `lib/features/expert/screens/expert_home_screen.dart` |

## 4. Implementation Details

We will add a consistent UI block at the bottom of the main scrollable content for each of the three home screens.

The UI block looks like this:

```dart
const SizedBox(height: 24),
Container(
  color: Colors.purple.shade50,
  padding: const EdgeInsets.all(16),
  child: Column(
    crossAxisAlignment: CrossAxisAlignment.start,
    children: [
      const Text(
        '🎥 Video Call Demonstration',
        style: TextStyle(
          fontSize: 18,
          fontWeight: FontWeight.bold,
          color: Colors.purple,
        ),
      ),
      const SizedBox(height: 12),
      SizedBox(
        width: double.infinity,
        child: ElevatedButton.icon(
          onPressed: () => context.push('/demo-video-call'),
          icon: const Icon(Icons.video_camera_front),
          label: const Text('Mở màn hình Video Call Demonstration'),
          style: ElevatedButton.styleFrom(
            backgroundColor: Colors.purple,
            foregroundColor: Colors.white,
            padding: const EdgeInsets.symmetric(vertical: 12),
          ),
        ),
      ),
    ],
  ),
),
```

### Specific Injection Points:

1.  **Member**: `lib/features/member/screens/home_screen.dart`
    - Inject just before the `const SizedBox(height: 90), // Space for bottom nav` towards the end of the `MemberHomeScreen` build method.
    - Ensure `go_router` is imported.

2.  **Rescuer**: `lib/features/rescuer/screens/rescuer_home_screen.dart`
    - Inject below the `_buildQuickAccess()` call in the `_HomeTabState` build method.
    - Add `import 'package:go_router/go_router.dart';` if missing.

3.  **Expert**: `lib/features/expert/screens/expert_home_screen.dart`
    - Inject below the final `_buildConsultationCard` (or similar list items) inside the `SliverList` of `_HomeTabState`. The injection should occur before `const SizedBox(height: 100),`.
    - Add `import 'package:go_router/go_router.dart';` if missing.

## 5. Validation Plan

- **Member App**: Navigate to the Home Screen. Scroll to the bottom and verify the "Video Call Demonstration" block appears. Tapping the button should navigate to the Demo Video Call Screen without breaking the back navigation stack.
- **Rescuer App**: Repeat the above steps.
- **Expert App**: Repeat the above steps.

## 6. Component Locator (For Future Modifications)

To save time searching the codebase in the future (e.g., when removing this temporary UI for Task 2), here are the exact locations of the newly implemented "Video Call Demonstration" UI blocks:

1. **`lib/features/member/screens/home_screen.dart`**
   - **Target**: The `Container` with the `Colors.purple.shade50` background.
   - **Location**: Inside `MemberHomeScreen`'s `build` method, located just before the `const SizedBox(height: 90), // Space for bottom nav` comment.

2. **`lib/features/rescuer/screens/rescuer_home_screen.dart`**
   - **Target**: The `Container` with the `Colors.purple.shade50` background.
   - **Location**: Inside `_HomeTabState`'s `build` method, located exactly below the `_buildQuickAccess(),` function call.

3. **`lib/features/expert/screens/expert_home_screen.dart`**
   - **Target**: The `Container` with the `Colors.purple.shade50` background.
   - **Location**: Inside `_HomeTabState`'s `build` method within the `SliverList` delegate, located after the `hasImage: false, ),` block and just before `const SizedBox(height: 100),`.
