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
* Best devlopment platform for Hot Reload and Hot Restart
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
* Notify user when the connection is lost
* Automatically recovers lost connection

### Special Android Settings
* Wi-Fi and Wake Locks
* Android Doze Mode Compliant
* Restart on device reboot
* Automatically turn on screen for configured alarms
* Automatically unlock device for configured alarms
* Persistant background service
* Single connection to the server
  * Message bus between the UI and Background Service
  * UI subscriptions are cleaned up when the UI is hidden

## Localization Support
* All user facing strings are in a localization file. Just need to add the translations for the other languages.
* Fully translated for Spanish

## Audio
* MP3 and WAV sound type support
* Basic ringtone support using the Ringtone audio stream
* Small collection of ringtones with Creative Commons Zero (CC0) license
* Short vibration when the device is in a call (Android/iOS)
* Connection lost audio 
* Queueing and priority for multiple ringtones
* Ability to upload/update/remove custom ringtones
* Locked factory default ringtones
* Max size restriction - 5 MB
  * Server and client enforced
* Max duration restriction - 20 seconds
  * Server enforced
* Server verifies valid sound files uploaded
* Ability to play, pause, stop any ringtone with slider
* Ability to rename or upload a new sound without changing existing configuration
* Sound data and metadata is cached on the clients, simple update id version determines if anything needs to be updated
  * Each sound also has a version id so that the client will only sync the changes
* Ringtone sound stops or is removed from queue if the underlying notification is no longer relevant
  * e.g. The notification ends or a user accepted the notification

## Local Notification Support
* Local notifications for configured alarms
* Opening alarms from local notifications
* Local notification are automatically kept in sync to show only the user notification that require action (Android/iOS)
  * If notification is accepted, ended, completed, expired, the local notification is removed
* Grouping multiple notification (Android)

## FullScreen Notification View
* This view can be automatically opened from any current view or background.
* The current notification content details are visible
* User can scroll through any other currently active notification
* There is a total active notifications indicator that shows all the current active notification by their icons
  * This indicator will show you which notification is currently being view in relation to the full active list.
     * This is done by making the icon for the currently viewed notification larger than the others.
  * This indicator will animate the notification icon that matches the current ringtone that is audible.
  * The notifications are sorted by priority
* Notifications that are opened automatically in FullScreen mode are not immediately marked as view by the client
  * This is because we don't know if the user is actually looking at the device sceen
  * The notification will be marked as view after the user performs some action (including exiting the view)

## Core Shared Packages
* Abstraction for logging in any package with verbose and other logs implemented across the system
* Extensions and other helpers

## Logging
* Using logging service for client app
* Open Telemetry Standard
* Can easily export the logs to Grafana(currently using Seq)
* Easily track all the information about any connected device
* Tracks all failed API request

## Client Syncing Data or Reconnect
* All relevant client data is syncronized after reconnect. Including new, updated, removed ringtones, pending notifications, patient data, assignments, ...etc. Almost all UI is refreshed.
  * Smart refresh. The data is syncronized only if the relevant service restarts. Restarts of unrelated services does not cause a data sync.
* Local notifications are syncronized after reconnecting to the services.
* Any notifications that occurred during disconnect period will be alerted to the user after reconnect.
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
  * Notify - notify to the users base on the configuration
  * Disabled - do not notify to anyone
  * Public Announcement - announce to all users
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
* If the notification is accepted, the reminder will be sent just to the users the accepted the notification.
* Reminders generate a local notification with a reminder sound.
  * The local notification will show the same data as the original notification but with a "Reminder:" text.
* Clicking on local notification will open the original notification that is being reminded.
* The original notification will animate when the reminder sound is playing.
* Reminders can also be generated by the source system without requiring any additional configuration.
  * Reminders from the source system are ignored if a reminder interval is already configured by the notification profile.
* The reminder local notification is removed automatically when it is no longer associated to the user.
  * This can happen when the notification is no longer active or someone else accepted the notification

## User Topology Management
* User is automatically shown the locations specific to their roles
* User is automatically drilled down to show locations if they dont have multiple unit or facility access

## Manage Subscriptions
* Each role change is effiecently updated in the backend for each click
* User is limited to 150 different subscription topics

## UI
* Persistant navigation even for app restarts
* Automatic UI scaling from small phone to full desktop monitor
* Can navigate most UI using keyboard instead of mouse
* Light, Dark and System themes supported
* Proper loading indicator for each action, disabling action waiting for response, global error/snackbar message queue
* Entensible custom UI
  * Numeric input control with validation, acceleration, default text options
  * Network related error UIs indicator
  * Snackbar messages and errors queues that handles message overloads
* Relative and absolute timestamps with dynamic dates base on how long the notification occured.
  * Tooltips to show the full relavant time
  * Relative time automatically updates
  * Localized with plurals vs singulars
* Basic patient alarm notification just for visualization
  * Improved horizontal scrolling with fade to indicate more scrollable data
* Custom UI for each notification type
  * Undeliverable notification can show the full notification details of the undelivered notification
  * The undeliverable notification, only has a reference to the orginal notification and take up minimal storage
* UI is automatically updated when the underlying data is changed on the backend.
  * This includes automatically updating the UI after reconnecting with the relevant service.
  * This includes all actions, additions, updates and deletes
  * This is implemented for the following screens:
    * Notification Profiles List
    * Specific Notification Profile Details
    * Category Alert Settings
    * All Sounds
    * Specific Sound
    * Notification System Settings
    * Active Notifications List
    * Notification Details View
    * Notification History
    * Patient Admit/Update/Discharge
    * Public Announcements
    * Custom Categories

## Observability (Shared Repository Pattern)
* For always in sync data there is a shared repository concept
* Different components can access any repository they care about
* Multiple components that need the same repository will retrieve the same one from the active pool
* Repositories will listen to changes and update the cache with just the updates (most of the time) and broadcast the new cache to the requesting components.
* Repositories will sync all the data after any reconnects to the notification broken or when the relevant service is restarted.
* Repositories have a pattern where data refresh/sync only occurs after a relevant service is reconnected.
* Pattern to check to memory leaks for repository that are not released.

## MTQQ 5 Client Support
* Works using TCP on Native Platforms
* Uses websockets for Web

## Integration Tests with Real Server or Mocks
* Working automation pipeline
* Automation code coverage report for the entire system (backend and frontend)
  * Frontend Coverage: 64.5% (8320 out of 12897 actionable lines)
  * Backend Coverage: 69.7% (10626 out of 15239 coverable lines)
* End to end automation test cases for receiving notifications
* Ability to test any platform with same automation test cases
  * **Web**: Safari, Chrome,
  * **Desktop**: Windows, Linux, MacOS
  * **Android**: Any device type or API level
  * **iOS**: Any OS level
* Intuitive automation pattern
  * Each screen has an associate automation test help that helps with all navigation/actions related to that screen
  * Very easy to read and maintain
  * [Sample Test](samples/automation_notification_settings.dart)

## Simple Frontend Setup
* The web front end automatically knows about all the endpoints, no setup required.
* The mobile/native implementation just need the initial setup configuration endpoint
  * Only the hostname is required, the app automatically distinguishes between production environment and development. Including https or http.

## Using JWT Tokens for security
* Automatic refresh of access token when it expires
  * Each API request is automatically intercepted and if we get a 401, then we automatically refresh the access token, and retry the orginal API request
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
  * Including configurable initial delay for any notification
* Ability to name/rename notification profiles
* Ability to delete notification profiles
* Ability to easily search/find any category by name

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

## Topology Notification Settings Screen
* Keep track of the available roles
* Automatically assign roles if names are the same
* Reuse one or more notification profile configuration across multiple Units (or Facilities)

## Notification Status/History
* Comprehensive tracking of all notification states and events with the ability to view the full history of any notification.
* Each status update includes the timestamp, action, update source
* Custom data available for each status update
  * i.e. For the sending status update we include the users to whom the notification was sent, escalation level of the notification
* Two different views for showing the notification history
  1. Quick view summary report
  2. Show all status updates individually
* Orange color to show escalation and progression of the notification
* Green to show positive actions such as accept or condition ended
* The ability to reset the notification after it was accepted
  * This will cause the notification to realarm and a new user will be able to accept the notification
* The ability to manually complete (end) the notification by the user (if allowed by notification profile/category configuration)
* Ability to view the notification status for each location level
  * i.e. If the same notification is alerted at both the facility level and room level

### Possible Notification Statuses
* User related statuses:
  - Sending → Delivered → Viewed → Accepted → Completed
* Other included statuses:
  - Escalation
    - by the User
    - automatic escalation because there are no users at the current escalation level
    - automatic escalation because the notification was not delivered to any device at the current escalation level
    - automatic escalation because no user accepted the notification within the configured time period
  - Undeliverable
    - Location not found
    - No notification profile exists
    - Unable to deliver to any end device
  - Delayed
  - Expired
  - Reset - the notification was reset by the user who accepted it
  - Disabled - the notification is disabled in the configuration
  - Reminder - a reminder of the active notification
  - Ended - by the source system

## Notification List Views
* The high level state of each notification can be view from the list view
  * Active, accepted, inactive, undeliverable
* The notification icon animates when the ringtone for that specific notification is being played
* The user has two list of notifications
  1. Active - Notifications that the user needs to Accept or Escalate
  2. Active Responsibilities - Notification that the user has Accepted but are still active
    - When the notification is ended, completed, expires, or reset, it is removed from the list of responsibilities.

## Custom Notification Categories
* Ability to create/delete custom notification categories.
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
* Gives user an indicator from other screens that there are pending notification that need to be handled
* Shows loading indicator then '?' if we cannot reach the server

## Simple Patient Management Solution
* Ability to Admit and Discharge a patient from locations
* Update existing patient details (First, Last, DOB, MRN, SSN, Other Id)
* Calendar picker for DOB
* Automatically refresh the details of the currently viewed patient if the backend changes

## Create Code Alerts
* Ability to create code alert from the frontend
* Swipe right to left anywhere to bring up additional menu from which you can select to create code alert
* On non-mobile devices, this menu can be brought up using Ctlr + M
* Automatically populates the lowest location that the current user has access to.
* Can select to create the alert notification for the enterprise, facility, unit, or room location level.
* Can add a custom message to the alert
* Select the alert code type by the color

## Patient Dashboard
* View the assigned locations and the patients assigned to that location
* Overview of the total number of active notifications for each location
* Indicator if each notification is accepted or there are some pending notifications
* All the active notification are being efficiently track for multiple locations. It is a matter of designing the UI to show the active notification for each location at the same time.

## Location/Patient Notifications
* View the currently active notifications for any location
* View the last 7 day history of all notifications for a specific location
  * Note: Can modify this to be patient specific and not location based

## Prevent Closing of Application
* Android/iOS - Continues to function in background when logged in
* Linux/Windows - Prevents user from easily closing the app without confirming log off

## One App for Notification App and Configuration
* Depending on the roles available for the user logging in, they will either see the notification app, configuration app, or both.

## Web Version Info
* Only one tab can be active at a time
* Additional tabs will show an message indicating any other active tabs need to be closed before the new tab can function
* Once the other tabs are closed, the new tab will automatically refresh to the same URL that was requested
* Local notificaton permission is requested by the browser
* WASM support

## Targeting Latest SDK versions
* Android 15 (SDK 35)
* iOS 18
* Flutter 3.32.1
* All package references are recent

## Test Tool
* Ability to one click to setup/config a new system. User is ready to receive notifications with a single click.
  * Generate a default notification profile
  * Assign the profile to the unit
  * Map the unit roles to the notification profile roles
  * Assign the default user the notification role for a room to be able to receive notifications
  * Admit a default patient to the room
* Ability to generate any category of notification
  * Ability to select/limit which category of notification should be generated
* Ability to generate a notification for any location within the system
* Ability to open the detailed notification view and status history for any generated notification
* Ability to track and end any generated notification
* One click performance testing
  * This sets up the notification profile with multiple roles, escalation levels and different settings for a few category of notification.
  * Then assigns that profile to all the units and maps the profile roles to the real roles of the unit.
  * The test tool divides all the locations among the configure number of clients.
  * Each client is setup:
    * Authenticates
    * Clears old assignments
    * Retrieves the avialable roles for the assigned location(s)
    * Assigns/Subscribes to all the roles for that assigned location(s)
    * Connects to the MQTT broker
    * Subscribes to the relevant topics
    * Sends delivered response to each received notification
    * Accepts each received notification 50% of the time
  * A simple report is generated to show the total number of sent notifications and the number of notifications received by the clients
* Client side logs including option to enable verbose logs are displayed by the test tool.

## Developer Notes
* Verbose logs when debugging
* Verifies repository resources are properly released in debug mode (memory leak check)

# Other Notes
* Dart uses code generation to support serialization.
* Doesn't handle polymorphism for swagger/openapi definitions
* No local(offline) demo mode (but we can instead host public test server for the sales folks)
* Weak Multi-Screen support for Windows - Would need to write native Windows code to support this
* Poor stacktraces on Web in release mode