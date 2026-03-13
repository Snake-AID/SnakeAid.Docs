# MAJOR FEATURES SUMMARY TABLE - SNAKEAID PLATFORM (ENGLISH VERSION)

## Project Information
- **Project Name:** AI-Powered Platform for Snakebite First Aid and Rescue Support (SnakeAid)
- **Purpose:** Integrated platform for snakebite first aid support, AI-powered snake species identification, rescue tracking, expert consultation, and incident monitoring

---

## Change Tracking

> [!IMPORTANT]
> This document now uses inline change tracking for migration support.
>
> [Current]
> The new business direction introduces a center-operated model with `Operator`, shift-aware rescuer assignment, single-center operations, and reduced medical-treatment scope.
>
> [Legacy]
> Previous requirements emphasized broader member self-service, direct rescue dispatch patterns, hospital/antivenom discovery, and a more open rescuer operating model.
>
> [Migration Impact]
> When reading a changed section, always treat `Current` as the source of truth and `Legacy` as context for backend migration.

# Major Features (English)

## Mobile Application for Member

> [!IMPORTANT]
> Changed Requirement
>
> [Current]
> Member flows remain the main entry point, but the system focuses on rescue support and incident handling rather than full medical-treatment management.
>
> [Legacy]
> The member app previously covered a larger treatment-oriented scope, including deeper hospital and antivenom support.
>
> [Migration Impact]
> Member-facing APIs related to treatment facilities and antivenom visibility should be reviewed for deprecation or external-reference-only handling.

### Emergency First Aid Guidance

- **Primary Actors:** Member
- **Features:**
  - FE-01: Provide step-by-step guidance when bitten by a snake (compression bandage, treatment methods, prohibited action warnings).
  - FE-02: Display proper compression bandage instructions with illustrated images.
  - FE-03: Warn against dangerous actions to avoid (applying leaves, cutting wound, sucking venom...).

### Emergency Rescue Call (SOS)

- **Primary Actors:** Member
- **Features:**
  - FE-04: SOS button sends GPS location and directly calls emergency hotline.
  - FE-05: Automatically share real-time location with assigned rescue team.

### Locate Nearest Treatment Facility

- **Primary Actors:** Member
- **Features:**
  - FE-06: Display map of hospitals/medical stations with snake antivenom.
  - FE-07: Calculate distance and estimated time to each facility.
  - FE-08: Detailed information about available antivenom types at each facility (based on admin-updated data).

### Track Bite and Symptoms

- **Primary Actors:** Member
- **Features:**
  - FE-09: Allow users to input symptom descriptions (pain, swelling, numbness, nausea...).
  - FE-10: Take photos of bite to track progression over time.
  - FE-11: Store symptom history and photos to provide to doctors or experts.

### Identify Snake Species from Image (AI)

- **Primary Actors:** Member
- **Features:**
  - FE-12: Use AI to identify snake species from photos.
  - FE-13: Display results: snake name, toxicity (venomous/non-venomous), danger level.
  - FE-14: Suggest appropriate first aid measures based on identified snake species.

### Assess Severity Level (AI)

- **Primary Actors:** Member
- **Features:**
  - FE-15: Analyze bite photos and symptoms to assess danger level.
  - FE-16: Issue emergency warning and recommend calling emergency services if critical.
  - FE-17: Classify severity levels: mild, moderate, severe, critical.

### Report Snakebite Incident / Snake Sighting

- **Primary Actors:** Member
- **Features:**
  - FE-18: Send GPS location report and snake images to system.
  - FE-19: Request snake rescue team to capture/relocate snake.
  - FE-20: Alert community about areas with venomous snake presence.

### Snake Prevention Knowledge

- **Primary Actors:** Member
- **Features:**
  - FE-21: Provide articles, videos about snake bite prevention.
  - FE-22: FAQ - Frequently asked questions about snakebite treatment.
  - FE-23: Information about common snake species in each area.

### Track Rescue Team Real-time

- **Primary Actors:** Member
- **Features:**
  - FE-24: Display rescue team's moving location on map after requesting rescue.
  - FE-25: Receive notifications when rescue team is en route and when mission is completed.
  - FE-26: Display route and estimated arrival time.

### Service Payment Management

- **Primary Actors:** Member
- **Features:**
  - FE-27: Pay online snake expert consultation fees.
  - FE-28: Pay snake rescue fees directly to rescue teams via platform.
  - FE-29: Track payment status and electronic invoices.
  - FE-30: View transaction history and used service details.

## Mobile Application for Snake Rescuer

> [!IMPORTANT]
> Changed Requirement
>
> [Current]
> Rescuers are staff of a single rescue center, receive accounts directly, work by shift, and are assigned missions through center operations.
>
> [Legacy]
> Previous requirements allowed more direct request-picking or system-led dispatch behavior in the rescuer flow.
>
> [Migration Impact]
> Backend must support staff onboarding, shift tracking, online status, and operator-driven assignment instead of relying only on member-rescuer direct dispatch patterns.

### Receive Snake Rescue Alerts

- **Primary Actors:** Snake Rescuer
- **Features:**
  - FE-01: Receive notifications about snake sightings or rescue requests with location and images.
  - FE-02: View request details: predicted snake type, danger level, contact information.

### Quick Confirmation and Response

- **Primary Actors:** Snake Rescuer
- **Features:**
  - FE-03: Confirm snake type (venomous/non-venomous) from images.
  - FE-04: Update verification results to system.
  - FE-05: Send preliminary information to requester about danger level.

### Rescue Task Management

- **Primary Actors:** Snake Rescuer
- **Features:**
  - FE-06: Accept or decline rescue requests.
  - FE-07: Update progress (moving, processing, completed).
  - FE-08: Manage task list: pending, in progress, completed.

### Safe Snake Capture Guidance

- **Primary Actors:** Snake Rescuer
- **Features:**
  - FE-09: Standard procedures for safely capturing and relocating snakes.
  - FE-10: List of necessary equipment and handling techniques for each species.
  - FE-11: Video tutorials for snake capture in various situations.

### Communication with Experts

- **Primary Actors:** Snake Rescuer, Snake Expert
- **Features:**
  - FE-12: Exchange information with snake experts for accurate identification.
  - FE-13: Request remote support when encountering difficult-to-identify snake species.
  - FE-14: Share real-time photos/videos with experts.

### Record and Report Activities

- **Primary Actors:** Snake Rescuer
- **Features:**
  - FE-15: Record detailed rescue cases (location, time, snake species, results).
  - FE-16: Take photos of captured snakes to store in database.
  - FE-17: Create statistical reports on rescue activities by month/quarter.

### Map Tracking and Navigation

- **Primary Actors:** Snake Rescuer, Member
- **Features:**
  - FE-18: Update rescue team's real-time location to system.
  - FE-19: Support navigation to member's location.
  - FE-20: Send status notifications (en route, arrived, completed) to member.

### Snake Identification from Image (AI)

- **Primary Actors:** Snake Rescuer
- **Features:**
  - FE-21: Use AI to identify snake species from member-submitted images.
  - FE-22: Receive warnings about danger level before arriving at scene.
  - FE-23: Prepare appropriate equipment and safety measures.

### Rescue Revenue Management

- **Primary Actors:** Snake Rescuer
- **Features:**
  - FE-24: Accept paid rescue requests from members.
  - FE-25: Track revenue, payment status, and transaction history.
  - FE-26: Receive payment via platform after rescue completion.
  - FE-27: View ratings and receive customer feedback to improve priority ranking.

## Operator / Dispatch Web Console

> [!IMPORTANT]
> New Requirement
>
> [Current]
> A new `Operator` role receives requests first, verifies incidents with the Member, checks rescuer online/shift status, assigns missions, and monitors rescue operations on the map.
>
> [Legacy]
> This role did not exist explicitly in the older requirement set and its responsibilities were split across system automation, Admin workflows, or direct rescuer actions.
>
> [Migration Impact]
> Backend will need operator-facing queues, verification states, assignment actions, rescue ping handling, and monitoring endpoints.

## Mobile Application for Snake Expert

> [!IMPORTANT]
> Changed Requirement
>
> [Current]
> Expert involvement remains optional and can include external snake experts providing professional consultation.
>
> [Legacy]
> Older requirements described experts more as tightly integrated platform actors without explicitly framing external participation.
>
> [Migration Impact]
> Review expert onboarding, availability, and consultation ownership rules.

### Verify Identification Data

- **Primary Actors:** Snake Expert
- **Features:**
  - FE-01: Confirm snake species from images/descriptions submitted by system or users.
  - FE-02: Modify AI results if identification is incorrect.
  - FE-03: Add professional notes on identification characteristics.

### Support AI Snake Identification

- **Primary Actors:** Snake Expert
- **Features:**
  - FE-04: Use AI to shorten snake species verification time.
  - FE-05: Review and approve AI results before publication.
  - FE-06: Train and improve AI model by confirming new data.

### Update First Aid Guidelines

- **Primary Actors:** Snake Expert
- **Features:**
  - FE-07: Update treatment and first aid procedures for each snake species.
  - FE-08: Compile detailed guidelines on symptoms and snake venom treatment methods.
  - FE-09: Provide information on appropriate antivenom dosages.

### Remote Consultation

- **Primary Actors:** Snake Expert, Member, Snake Rescuer
- **Features:**
  - FE-10: Provide online support for members via chat/video call.
  - FE-11: Consult rescue teams on handling complex snake species.
  - FE-12: Assess member condition and recommend emergency measures.

### Consultation Revenue Management

- **Primary Actors:** Snake Expert
- **Features:**
  - FE-13: Set online consultation fee rates.
  - FE-14: Receive payment via platform and issue electronic invoices.
  - FE-15: View revenue reports by month/quarter.
  - FE-16: Track consultation sessions and customer reviews.

## Admin Web Application

> [!IMPORTANT]
> Changed Requirement
>
> [Current]
> Admin / Manager focuses on governance, configuration, reporting, and oversight, while frontline dispatch responsibility shifts to Operator.
>
> [Legacy]
> Older requirements grouped more operational dispatch responsibility under Admin and included treatment-facility / antivenom management as active platform modules.
>
> [Migration Impact]
> Separate managerial controls from operator workflows. Hospital and antivenom modules should be marked legacy unless explicitly retained as external reference data.

### User and Permission Management

- **Primary Actors:** Admin
- **Features:**
  - FE-01: Create accounts for members, experts, and rescue teams.
  - FE-02: Assign access permissions by role (Member, Rescuer, Expert, Admin).
  - FE-03: Manage account status (activate, lock, delete).
  - FE-04: View user activity history.

### Snake Species Database Management

- **Primary Actors:** Admin
- **Features:**
  - FE-05: Add, edit, delete snake species information (scientific name, local name, characteristics, distribution).
  - FE-06: Upload snake images for each species.
  - FE-07: Manage information about snake behavior and habitat.
  - FE-08: Classify snakes by danger level and distribution area.

### Treatment Facility Management

- **Primary Actors:** Admin
- **Features:**
  - FE-09: Add hospital/medical station information (name, address, GPS coordinates).
  - FE-10: Update list of available antivenom types at each facility.
  - FE-11: Manage operating hours and emergency contact information.
  - FE-12: Mark facilities capable of treating venomous snake bites 24/7.

### System Content Management

- **Primary Actors:** Admin
- **Features:**
  - FE-13: Update first aid guidelines, snake information, dangerous areas.
  - FE-14: Edit articles, educational videos on snake bite prevention.
  - FE-15: Manage FAQ and user help content.
  - FE-16: Post news updates on seasonal venomous snake situations.

### Data Statistics and Reporting

- **Primary Actors:** Admin
- **Features:**
  - FE-17: Compile snakebite cases by region, time, snake species.
  - FE-18: Report rescue operations and rescue team completion rates.
  - FE-19: Compile consultation statistics and expert reviews.
  - FE-20: Analyze trends and high-risk areas.
  - FE-21: Export reports by month/quarter/year.

### Community Alerts and Announcements

- **Primary Actors:** Admin
- **Features:**
  - FE-22: Send alerts about areas with frequent venomous snake appearances.
  - FE-23: Push seasonal prevention guidance notifications.
  - FE-24: Issue emergency alerts when new dangerous venomous snake species detected.
  - FE-25: Manage notification recipient list by area.

### Map Activity Monitoring

- **Primary Actors:** Admin
- **Features:**
  - FE-26: Display real-time ongoing rescue cases on map.
  - FE-27: Track rescue team locations and task status.
  - FE-28: View heat map of snakebite hotspots.
  - FE-29: Monitor average response time of rescue teams.

### Service Fee and Revenue Management

- **Primary Actors:** Admin
- **Features:**
  - FE-30: Set fee rates for rescue services and expert consultation.
  - FE-31: Track total revenue and distribute income to rescuers/experts.
  - FE-32: Manage payments between members – rescuers/experts – platform.
  - FE-33: Create periodic financial reports (monthly/quarterly/yearly).
  - FE-34: Manage platform commission and refund policies.
  - FE-35: Handle payment disputes and refund requests.

---

# Notes
- **FE:** Feature (Functional Feature)
- Each module starts from FE-01 for easier management and tracking
- Primary Actors include: **Member**, **Snake Rescuer**, **Snake Expert**, **Admin**
