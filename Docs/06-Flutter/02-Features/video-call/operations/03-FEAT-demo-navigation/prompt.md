---
doc_role: operation
operation_id: FEAT-demo-navigation
generated_from: plan.md
status: in_progress
created_at: 2026-02-25
---

# FEAT-demo-navigation — Flutter Prompt

## Objective

Add a temporary UI block to the bottom of the Member, Rescuer, and Expert home screens to navigate to the Demo Video Call Screen (`/demo-video-call`).

## Code Culture Rules (MUST FOLLOW)

- Follow existing codebase conventions strictly.
- Use `context.push('/demo-video-call')` for navigation to preserve the back stack.
- Ensure `import 'package:go_router/go_router.dart';` is present where `context.push` is used.

## Required Outputs

### 1. Update Member Home Screen

**File**: `lib/features/member/screens/home_screen.dart`

Add the following block to the end of the `SingleChildScrollView` content in `MemberHomeScreen`, just above the `SizedBox` reserved for the bottom navigation bar:

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

### 2. Update Rescuer Home Screen

**File**: `lib/features/rescuer/screens/rescuer_home_screen.dart`

Ensure `package:go_router/go_router.dart` is imported.

Add the same UI block to the `_HomeTabState` build method's `SingleChildScrollView`, immediately following the `_buildQuickAccess()` method call placeholder.

### 3. Update Expert Home Screen

**File**: `lib/features/expert/screens/expert_home_screen.dart`

Ensure `package:go_router/go_router.dart` is imported.

Add the same UI block to the `_HomeTabState` build method's `SliverList`, placing it after the list of consultations and just before the spacing block at the end (`SizedBox(height: 100)`).
