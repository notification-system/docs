# Vital Echo Overview

## Platform Support
* Android
* iOS
* Web
* Windows
* Linux
### Potential Support
* *MacOS* - Needs more testing. Likely requires additional entitlements or custom code for some native functionalities.
* *Embedded Devices(Monitors)* - Potential to share some UI and packages across embedded devices.

Ability to access native/unique features for each platform when necessary.

## Developer Experience
* Best development platform for Hot Reload and Hot Restart
  * Instantaneously apply new code updates to the currently running app
  * Fully develop new features using the same debug connection without recompiling the solution
* Simultaneously debug the app on Desktop, Mobile and Web
* Run the automation tests for any platform

## Deep/Web Linking (Android/iOS/Web)
* Url for WebApp and Schema for Android/iOS (Can be converted to universal links support)
  * The same path and query parameters work for Android/iOS/Web
* Every screen is accessible using URI automatically including parameter options
* Stores the pending deep/web link request to be completed after successful login.
  * Clears the link if the app is inactive in between
* Allows for configuration where a single click of a dedicated button on hardened devices can be easily linked to any action in the app

### View Notification History
* Access the notification data and history for any notification by id
  * Example: https://host/#/notifications/[id]

## REST APIs
* Auto generated code for the data models and API's to keep the server and client in sync
* Pattern to display any Rest API errors with ability for the user to retry
  * Most screens also auto refresh content on reconnect to MQTT or when the relevant service restarts.
* Network related error codes are warnings. Other http status codes logged as errors.

## Reliable Persistant Connection (Android/Linux/Windows. Foreground only on iOS for now)
* CDAS compliant implementation
* Notify user when the connection is lost with a disconnect sound and visual
  * There is a 10 second grace period where the disconnect sound is not played even if the connection is detected as disconnected.
    * This allows the client to recover and not disrupt the user for short disconnects (especially for environments with poor roaming setup)
* Automatically recovers lost connection and re-syncs all relevant data (including alarming for any missed notifications)

## Audio/Sounds
* MP3 and WAV sound type support
* Basic ringtone support using the Ringtone audio stream
* Locked factory default ringtones
  * Small collection of ringtones with Creative Commons Zero (CC0) license
* Short vibration when the device is in a call (Android/iOS)
* Connection lost audio 
* If multiple notifications are received simultaneously, then the ringtone for each notification will be played one after the other.
  * If the notifications have different configured priorities, the ringtone queue is sorted by that priority order
* Ringtone sound stops or is removed from queue if the underlying notification is no longer relevant
  * e.g. The notification ends or a user accepted the notification

### Custom Sounds
* Ability to upload/update/remove custom ringtones
* Max size restriction - 5 MB
  * Server and client enforced
* Max duration restriction - 20 seconds
  * Server enforced
* Server verifies valid sound files uploaded
* Ability to play, pause, stop any ringtone. 
  * Editable slider that indicates the duration of the sound and the current position.
* Ability to rename or upload a new sound without changing existing configuration
* Sound data and metadata is cached on the clients, simple update id version determines if anything needs to be updated
  * Each sound also has a version id so that the client will only sync the changes
* Audio will fallback to the default sound if a configuration exists for a deleted custom sound

## Local Notification Support
* Local notifications for configured alarms
* Opening alarms from local notifications
* Local notification are automatically kept in sync to show only the user notifications that require action (Android/iOS)
  * If notification is accepted, ended, completed, or expired, then the local notification is removed
  * This also includes syncing the local notification state after a reconnect to the server
* Grouping multiple notification (Android)

## FullScreen Notification View
* This view can be automatically opened from any current view or background.
* The current notification content details are immediately visible
* User can scroll through any other currently active notification
* There is a total active notifications indicator that shows all the current active notifications by their icons
  * This indicator will show you which notification is currently being viewed in relation to the full active list.
     * This is done by making the icon for the currently viewed notification larger than the others.
  * This indicator will animate the notification icon that matches the current ringtone that is audible.
  * The notifications are sorted by priority
* Notifications that are opened automatically in FullScreen mode are not immediately marked as view by the client
  * This is because we don't know if the user is actually looking at the device screen
  * The notification will be marked as view after the user performs some action (including exiting the view)

## Client Syncing Data or Reconnect
* All relevant client data is synchronized after reconnect. Including new, updated, removed ringtones, pending notifications, patient data, assignments, ...etc.
  * Smart refresh. The data is synchronized only if the relevant service restarts. Restarts of unrelated services does not cause a data sync.
* Local notifications are synchronized after reconnecting to the services.
* Any notifications that occurred during disconnect period will be alerted to the user after a reconnect if they are still active.
  * This will be according to the configured ringtone and alert settings of each particular notification.
      * i.e. If the synced notification was full screen, it will alarm as full screen after reconnect.

## Frontend Storage
* Simple key value pair storage with observable data
* Native secure storage solutions

## Shared Data Model Auto Generation
* OpenApi generation of the client Api and DTO Models

## Notification Settings (per Notification Category)
Per notification category the following notification settings are possible:
* Behavior:
  * Notify - notify to the users based on the configuration
  * Disabled - do not notify to anyone
  * Public Announcement - announce to all users
  * Notify and Announce - combination of announcing to all users and also notifying to the users based on the configuration so that the notification can be handled
    * Conflict logic on the client to prioritize the direct notification instead of the public announcement if the client receives both.
      * i.e. Only one local notification is generated and tapping the local notification will navigate to the notification for the user and not the public announcement.
* Ringtone to play
* Priority (used for audio and visual priority)
* Minimum volume level (Android, Windows, Linux, partial iOS)
* Turn on the screen (Android only)
* Unlock the device (Android only)
* Notification Visual:
  * Fullscreen - Opens directly to user (Android/Windows only)
  * Local Notification 
* Vibrate (Android/iOS)
* Can the notification be manually completed by the accepted user
* Reminder Interval
* Expiration

## Reminder Updates
* Ability to configure periodic reminders for active notifications per category of notifications.
* If the active notification is not accepted by anyone, the reminder will be sent to all associated users.
* If the notification is accepted, the reminder will be sent just to the users who accepted the notification.
* Reminders generate a local notification with a reminder sound.
  * The local notification will show the same data as the original notification but with a "Reminder:" text.
* Clicking on local notification will open the original notification that is being reminded.
* The original notification will animate when the reminder sound is playing.
* Reminders can also be generated by the source system without requiring any additional configuration.
  * Reminders from the source system are ignored if a reminder interval is already configured by the notification profile.
* The reminder local notification is removed automatically when it is no longer associated to the user.
  * This can happen when the notification is no longer active or someone else has accepted the notification

## User Topology Management
* User is automatically shown the locations specific to their roles
* User is automatically drilled down to show rooms if they don't have multiple unit or facility access
  * i.e. If the user has access to just one unit, then they will only see the rooms for that unit.

## Manage Subscriptions
* Each role change is efficiently updated in the backend for each click
* User is limited to 150 different assigned roles

## UI
* Persistent navigation backstack even for app restarts
* Automatic UI scaling from small phone to full desktop monitor
* Can navigate the UI using keyboard instead of mouse
* Light, Dark and System themes supported
* Proper loading indicator for each action. Including disabling of relevant actions while waiting for response.
* Snackbar message/error queue
  * If multiple snackbar messages/errors need to display they will appear one after the other.
  * Control how long each message/error displays to the user
  * If there are more than 2 messages queued, the duration is shortened.
  * If there are more than 4 messages queued, the excess messages are grouped together.
* Standard confirmation popup UI for delete actions with custom UI(loading indicator) for async actions
* Extensible custom UI
  * Numeric input control with validation, acceleration, default text options
  * Network related error UIs indicator
* Inline validation and error messages for required fields
  * i.e. username, custom category fields, patient lastname, ... etc.
* Standard timestamps UI across the entire app that can be configured to show the absolute or relative timestamp
  * Tooltips to show the full relevant time (hover on desktop or tap on mobile)
  * Relative timestamps automatically update every 5 seconds
    * Localized with plurals vs singulars
* Basic patient alarm notification just for visualization
  * Improved horizontal scrolling with fade to indicate more scrollable data
* Custom UI for each notification type
  * Undeliverable notification can show the full notification details of the undelivered notification
  * The undeliverable notification, only has a reference to the original notification and takes up minimal storage
* All UI is automatically updated when the underlying data is changed on the backend.
  * This includes automatically updating the UI after reconnecting with the relevant service.
  * This includes all actions, additions, updates and deletes
  * This is implemented for the following screens:
    * Home Dashboard
    * Notification Profiles List
    * Specific Notification Profile Details
    * Category Alert Settings
    * All Sounds
    * Specific Sound
    * Notification System Settings
    * Authentication System Settings
    * Active Notifications List (including FullScreen Notifications)
    * Notification Details View
    * Notification History
    * Patient Admit/Update/Discharge
    * Public Announcements
    * Custom Categories
    * Location Notification Settings
    * Role Assignments Selection

## Observability (Shared Repository Pattern)
* For always in sync data there is a shared repository concept
* Different components can access any repository they care about
* Multiple components that need the same repository will retrieve the same one from the active pool
* Repositories will listen to changes and update the cache with just the updates (most of the time) and broadcast the new cache to the requesting components.
* Repositories will sync all the data after any reconnects to the notification Mqtt broker or when the relevant service is restarted.
* Repositories have a pattern where data refresh/sync only occurs after a relevant service is reconnected.
* Pattern to check for memory leaks for repository that are not released properly.

## MQTT 5 Client Support
* Works using TCP on Native Platforms
* Uses websockets for Web

## CI and Integration Tests
* Working automation CI pipeline
  * These are automated tests running against the client application, which is connected to a complete backend system.
  * Backend is tested on Linux, Windows and MacOS
  * Frontend is tested on Android, iOS, Web(Chrome) and Windows
* Automation code coverage report for the entire system (backend and frontend)
  * Frontend Coverage: 65.6% (8465 out of 12900 actionable lines)
  * Backend Coverage: 73.7% (11308 out of 15325 coverable lines)
* End to end automation test cases for receiving notifications
  * Including automation tests for system durability/recovery after service restarts
* Ability to test any platform with same automation test cases
  * **Web**: Safari, Chrome,
  * **Desktop**: Windows, Linux, MacOS
  * **Android**: Any device type or API level
  * **iOS**: Any OS level
* Intuitive automation pattern
  * Each screen has an associated automation test helper that aids with all navigation/actions related to that screen
  * Very easy to read and maintain
  * [Sample Test](samples/automation_notification_settings.dart)

### Additional CI/CD Details
* Workflows to validate backend and frontend builds with code analysis
* Workflow to generate the release web version of the main frontend application and the test tool application
* Workflow to generate release Android builds
* Workflow for one-click deployment of a Git branch of the full system onto a Linux machine using Production configuration
  * Currently uses Docker Compose
  * Option to clear all volumes/data or retain previous data after deployment
  * Note: An initial provisioning workflow must be executed once on the target Linux machine before deployment

## Simple Frontend Setup
* The Web version of the app automatically knows about all the endpoints within the system, no setup required.
* The Mobile/Native implementation just need the initial setup configuration endpoint
  * Only the hostname is required, the app automatically distinguishes between production environment and development. Including https vs http or ws vs wss.

## Using JWT Tokens for security
* Automatic refresh of access token when it expires
  * Each API request is automatically intercepted and if we get a 401, then we automatically refresh the access token, and retry the original API request
* Log off when refresh token expires
  * Plays disconnect ringtone and show log off reason to user

## Notification profile screen
* Expand to show all categories
  * Option to show only categories that have different settings than the parent category
* Ability to track and reset categories that have settings different from parent category
* Ability to add/remove escalation levels
  * Can configure as many escalation levels as needed
* Ability to export/import any notification profile settings
  * User will be prompted when importing notification profiles the conflict with existing profiles. User can cancel the operation or force override.
* UI indicator to show disabled notification
* Add/remove profile roles
* Assign multiple roles per escalation level
* Configure any notification setting per category
* Configurable delay between each escalation level
  * Including configurable initial delay for any notification category
* Ability to name/rename notification profiles
* Ability to delete notification profiles
* Ability to easily search/find any category by name
* Notification categories that are configured as public announcements only do not show the escalation level configuration (since it does not matter)

## Ability to Select Notification Profile per Patient
* Scenarios where different patient profiles can have different notification configurations
* This opens up ability to assign patients based on teams. A notification profile can really be just a team.
  * We can put patients without a team to a default team or have some orchestrator determine which team should take the patient
  * This could depend on which team has least assigned patients per online members of the team

## Public Announcement
* Public Announcements are announced to anyone associated with a particular location.
* This can be the Enterprise (public announcement to every single user), Facility level announcements, or Unit level announcements
* User can read all the previous public announcements for any of those locations
* Users with permission can create public announcements from the same screen where they read the existing announcements
* Selecting the local notification for the public announcement will navigate to the local announcement board for that location
* Ability to define any category of notification as a Public Announcement. Some notifications can be automatically made public announcements.
  * For example an evacuation Code Alert you might want to announce publicly to everyone.
  * The notification can still also alarm based on the escalation level
  * Handling scenario where the ringtone is played only once when a user receives an active notification they need to address and the public announcement for the same event.

## Location Notification Settings Screen
* Ability to setup the notification profiles for the Enterprise, Facility or Unit
* Ability to quickly search existing notification profile and assign it to the location
* Ability to assign/unassign the notification profile role to the actual role within a location.
  * Can assign multiple location roles for a single notification profile role
* The system will automatically assign roles if names are the same
  * i.e. If a "Nurse" notification profile role exists, it will auto map to the "Nurse" location role
* Select the default notification profile for the location if multiple profiles are assigned
* Allows the same notification profile to be used across multiple Units (or Facilities)

## Notification Status/History
* Comprehensive tracking of all notification states and events with the ability to view the full history of any notification.
* Each status update includes the timestamp, action, update source
* Custom data available for each status update
  * i.e. For the sending status update we include the users to whom the notification was sent, escalation level of the notification
* Two different views for showing the notification history
  1. Quick view summary report
    - Groups all status for one user into one row per escalation level
  2. Show all status updates individually
* Orange color used to show status updates that escalate or progress the notification
* Green to show positive actions such as accepted, condition ended or notification was marked as completed.
* The ability to reset the notification after it was accepted
  * This will cause the notification to realarm and a new user will be able to accept the notification
* The ability to manually complete (end) the notification by the user (if allowed by notification profile/category configuration)
* Ability to view the notification status for each location level
  * i.e. If the same notification is alerted at both the facility level and room level

### Possible Notification Statuses
* User related statuses:
  - Sending → Delivered → Viewed → Accepted → Completed
* Other included statuses:
  - Escalation (includes reason)
    - by the User
    - automatic escalation because there are no users at the current escalation level
    - automatic escalation because the notification was not delivered to any device at the current escalation level
    - automatic escalation because no user accepted the notification within the configured time period
  - Undeliverable (includes reason)
    - Location not found
    - No notification profile exists
    - Unable to deliver to any end device
  - Delayed (includes delayed period)
  - Expired
  - Reset - the notification was reset by the user who accepted it
  - Disabled - the notification is disabled in the configuration
  - Reminder - a reminder of the active notification
  - Ended - by the source system

## Notification List Views
* The high level state of each notification can be viewed from the list view
  * Active, accepted, inactive, undeliverable
* The notification icon animates when the ringtone for that specific notification is being played
* The user has two lists of notifications
  1. Active - Notifications that the user needs to Accept or Escalate
  2. Active Responsibilities - Notification that the user has Accepted but are still active
    - When the notification is ended, completed, expires, or reset, it is removed from the list of responsibilities.

## Custom Notification Categories
* Ability to create/delete/update custom notification categories.
* Custom categories can be a child of any factory or other custom categories.
  * Deleting parent custom category will delete all child custom categories.
* Custom categories can be configured separately for each notification profile, like any other category.
* Ability to configure the following for the custom categories:
  1. Id
  2. Name
  3. Parent Category
  4. Icon (SVG format) (Work in progress)
  5. Icon Color
  6. Animation Type
* The client maintains the custom categories and uses the configured icon, color and animation type.
* Required fields are indicated when attempting to save or update with missing data
* Validation of new category ids to make sure they don't interfere with reserved system ids and other categories
* Allows a customer to build a small middleware to parse any custom data and send the data to the notification system using a new specific category with unique notification settings. This will not require a new version of the system.

## Number of Active Notifications Indicator
* Provides the user an indicator from that there are pending notification that need to be handled
* Shows loading indicator then '?' if we cannot reach the server

## Simple Patient Management Solution
* Ability to Admit and Discharge a patient from locations
* Update existing patient details (First, Last, DOB, MRN, SSN, Other Id)
* Calendar picker for DOB
* Automatically refresh the details of the currently viewed patient if the backend changes

## Create Code Alerts
* Ability to create code alert from the frontend app
* Swipe right to left anywhere to bring up additional menu from which you can select to create code alert
* On non-mobile devices, this menu can be brought up using Ctrl + M
* Automatically populates the lowest location that the current user has access to.
* Can select to create the alert notification for the enterprise, facility, unit, or room location level.
* Can add a custom message to the alert
* Select the alert code type by the color

## Patient Dashboard
* View the assigned locations and the patients assigned to that location
* Overview of the total number of active notifications for each location
* Indicator if each notification is accepted or there are some pending notifications
* All the active notifications are being efficiently tracked for multiple locations. It is a matter of designing the UI to show the active notification for each location at the same time.

## Location/Patient Notifications
* View the currently active notifications for any location
* View the last 7 day history of all notifications for a specific location
  * Note: Can modify this to be patient specific and not location based

## One App for Notification App and Configuration
* Depending on the roles available for the user logging in, they will either see the notification app, configuration app, or both.

## Platform Specials
### Android
* Native Wi-Fi and Wake Locks are used
* Android Doze Mode Compliant
* Restart service on device reboot
* Automatically turn on screen for configured alarms
* Automatically unlock device for configured alarms
* Persistant background service
  * Stopped only when user is logged off
* Single connection to the server
  * Message bus between the UI and Background Service
  * UI subscriptions are cleaned up when the UI is hidden

### iOS
* Starts Local Push Provider service for persistant background connection
  * Stopped only when user is logged off

### Web Version Info
* Only one tab can be active at a time
  * Additional tabs will show a message indicating any other active tabs need to be closed before the new tab can function
  * Once the other tabs are closed, the new tab will automatically refresh to the same URL that was requested
* Local notification permission is requested by the browser
* WASM support

### Linux/Windows
* Prevents user from easily closing the app without log off

## Core Platform and Api Manager
* Packages that are shared between the main app and test tool
* Extensions and other helpers

### Logging
* Abstract logging interface in the core platform
* Logs data to the backend in Open Telemetry format
* If the backend is unreachable, first 10 and last 5 logs are retained and resent on next valid connection.
  * The resent logs are marked as resent.
* Can easily export the logs to Grafana
* Concept of verbose logs which are only enabled when debugging (Can map to a setting on the server)

## Localization Support
* All user facing strings are in a localization file. Just need to add the translations for the other languages.
* Fully translated for Spanish

## Targeting Latest SDK versions
* Android 15 (SDK 35)
* iOS 18
* Flutter 3.32.3
* All package references are recent

## Test Tool
* Ability to one click to setup/config a new system. User is ready to receive notifications with a single click.
  * Generate a default notification profile
  * Assign the profile to the unit
  * Map the unit roles to the notification profile roles
  * Assign the default user the notification role for a room to be able to receive notifications
  * Admit a default patient to the room
* Ability to generate notifications for any category
  * Ability to select/limit which category of notification should be generated
  * This includes any custom generated notification categories.
    * The UI automatically syncs with the notification categories defined by the server.
* Ability to generate a notification for any location within the system
  * This includes generating notification for the Room, Unit, Facility or Enterprise
* Ability to open the detailed notification view and status history for any generated notification
* Ability to track and end any generated notification
* One click performance testing
  * This sets up the notification profile with multiple roles, escalation levels and different settings for a few categories of notification.
  * Then assigns that profile to all the units and maps the profile roles to the real roles of the unit.
  * The test tool divides all the locations among the configured number of clients.
  * Each client is setup:
    * Authenticates
    * Clears old assignments
    * Retrieves the available roles for the assigned location(s)
    * Assigns/Subscribes to all the roles for that assigned location(s)
    * Connects to the MQTT broker
    * Subscribes to the relevant topics
    * Sends delivered response to each received notification
    * Accepts each received notification 50% of the time
  * A simple report is generated to show the total number of sent notifications and the number of notifications received by the clients at the end of the test.
  * Test tool automatically stops (with an error message) if the server or the test tool is unable to maintain the configured throughput.
* Client(test tool) side logs including option to enable verbose logs are displayed by the test tool.
  * Since the test tool partly shares the same code with the real app, this includes logs that would be output by the real app.
  * Logs are color coded based on the severity
  * In-memory logs are rotated after 1000 logs.

# Other Notes
* Dart uses code generation to support serialization.
* Doesn't handle polymorphism for swagger/openapi definitions
* No local(offline) demo mode (but we can instead host public test server for the sales folks)
* Weak Multi-Screen support for Windows - Would need to write native Windows code to support this