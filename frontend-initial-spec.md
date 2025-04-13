# Features Implemented

## Platform Support
* Android
* iOS
* Web
* Windows
* Linux
### Potential Support
* *MacOS* - Needs more testing. Likely requires additional entitlements or custom code for some native functionalities.
* *Embedded Devices(Monitors)* - Potential to share some UI and packages across embedded devices.

## Deep/Web Linking (Android/iOS/Web)
* Url for WebApp and Schema for Android/iOS (Can be converted to universal links support)
* Current schema for notificationapp:// and serverconfigapp://
* Every screen is accessible using URI automatically including parameter options
* Stores the pending deep/web link request to be completed after successful login.
  * Clears the link if the app is inactive in between
* Allow configuration where a single click of a dedicated button on hardened devices can be linked easily to any action in the app

## MTQQ 5 Client Support
* Works using TCP on Native Platforms
* Uses websockets for Web

## REST API Communication Package
* Auto generated code for the data models and API's
* User is notified when any rest API fails
* Network related error codes are warnings. Other http status codes are errors.

## Reliable Persistant Connection (Android/Linux/Windows)
* CDAS compliant implementation
* Notify user when the connection is lost/recovered
* Configurable delay when the connection is lost
* Handles automatic disconnects from the server.

### Special Android Settings
* Wi-Fi and Wake Locks
* Android Doze Mode Compliant
* Restart on device reboot
* Automatically turn on screen for configured alarms
* Automatically unlock device for configured alarms

## Localization Support
* All user facing strings are localized. Just need to add the translations for the other languages.

## Audio
* MP3 and WAV sound type support
* Basic ringtone support using the Ringtone audio stream
* Small collection of ringtones with Creative Commons Zero (CC0) license
* Short vibration when the device is in a call (Android/iOS)
* Disconnect local notification
* Queueing and priority
* Ability to upload/remove custom ringtones
* Max size and length restrictions
* Server verifies valid sound files uploaded
* Ability to play, pause, stop any ringtone with slider
* Ability to change the sound name
* Ability to upload a new sound without changing existing configuration
* Locked factory default notification types
* Sound data and metadata is cached on the clients, simple update id version determines if anything needs to be updated
* The client will only sync the changes

## Local Notification Support (Android/iOS/Linux/Windows)
* Local notifications for configured alarms
* Opening alarms from local notifications
* Grouping multiple notification (Android/iOS)

## Client Syncing Data or Reconnect
* All relevant client data is syncronized after reconnect. Including new, updated, removed ringtones and current pending notifications.
* Local notifications are syncronized after reconnecting to the services.
* Any notifications that occurred during disconnect period will be alerted to the user after reconnect.
  * This will be according to the configured ringtone and alert settings of each particular notification.
      * i.e. If the synced notification was full screen, it will alarm as full screen after reconnect.
* These notifications updates are also alerted to the user on login if for some reason data was sent to the user while they were offline.

## FullScreen Notification View
* This view can be automatically opened from any current view or background.
* The current notification content details are visible
* User can scroll through any other currently active notification
* There is a total active notifications indicator that shows all the current active notification by their icons
  * This indicator will show you which notification is currently being view in relation to the full active list.
     * This is done by making the icon for the currently viewed notification larger than the others.
  * This indicator will animate the notification icon that matches the current ringtone that is audible.

## Core Shared Packages
* Abstraction for logging in any package with verbose and other logs implemented across the system
* Extensions and other helpers

## Logging
* Using logging service for client app
* Open Telemetry Standard
* Can easily export the logs to Grafana or Seq
* Easily track all the information about any connected device
* Tracks all failed API request

## Frontend Storage
* Simple key value pair storage with observable data
* Native secure storage solutions (Web solution is experimental)

## Shared Data Model Auto Generation
* OpenApi generation of the client Api and DTO Models

## Notification Settings (per Notification)
Per notification category the following notification settings are possible:
* Behavior:
  * Notify - notify to the users base on the configuration
  * Disabled - do not notify to anyone
  * Public Announcement - announce to all users
* Ringtone to play
* Priority (used for audio and visual priority)
* Minimum volume level (partial support in iOS)
* Turn on the screen (Android only)
* Unlock the device (Android only)
* Notification Visual:
  * Fullscreen - Opens directly to user (Android/Windows only)
  * Local Notification 
* Vibrate (Android/iOS)
* Can the notification be manually completed by the accepted user
* Expiration

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
  * Numeric input control with validation, acceleation, default text options
  * Network related error UIs indicator
  * Snackbar messages and errors queues that handles message overloads
* Relative and absolute timestamps with dynamic dates base on how long the notification occured.
  * Tooltips to show the full relavant time
  * Relative time automatically updates
  * Localized with plurals vs singulars
* Basic patient alarm notification just for visualization
  * Improved horizontal scrolling with fade to indicate more scrollable data
* Custom UI for each notification type
  * Undeliverable notification can show the full undelivered notification details
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

## Observability (Shared Repository Pattern)
* For always in sync data there is a shared repository concept
* Different components can access any repository they care about
* Multiple components that need the same repository will retrieve the same one from the active pool
* Repositories will listen to changes and update the cache with just the updates (most of the time) and broadcast the new cache to the requesting components.
* Repositories will sync all the data after any reconnects to the notification broken or when the relevant service is restarted.
* Repositories have a pattern where data refresh/sync only occurs after a relevant service is reconnected.
* Pattern to check to memory leaks for repository that are not released.

## Integration Tests with Real Server or Mocks
* Simple integration test to make sure the app launches on web
* Automation testing code coverage report for the entire system (backend and frontend)
* Can easily run these same automation tests on wide range of devices
  * **Web**: Safari, Chrome,
  * **Desktop**: Windows, Linux, MacOS
  * **Android**: Any device type or API level
  * **iOS**: Any OS level

## Simple Frontend Setup
* The web front end automatically knows about all the endpoints, no setup required.
* The mobile/native implementation just need the initial setup configuration endpoint
  * Only the hostname is required, the app automatically distinguishes between production environment and development. Including https or http.

## Using JWT Tokens for security
* Automatic refresh of access token when it expires
* Refresh token management
* Log off when refresh token expires
  * Plays disconnect ringtone and show log off reason to user
* Only internal systems can publish notifications to the message bus
* External system required access token and their notification are verified before publishing
* Limit a single user to 40 roles
* Multiple or single user session configuration
* All public APIs are protected

## Notification profile screen
* Lots of goodies here
* Expand to show all categories
* Collapse to show only the modified categories
* Ability to export/import any notification profile settings (Windows/Mac/Web)
* Small UI improvements to show disabled categories
* Ability to track and reset categories that have settings different from parent category
* Removed absract roles are also remove from any assignments

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

## Notification Statuses
* Delivered status for each delivered notification
* Seen status when the user manually opens the notification details view
* Delivery summary report vs showing all status updates individually
* Orange color to show escalation and progression of the notification
* Green to show positive actions such as accept or condition ended
* The ability to reset the notification after it was accepted
  * This will cause the notification to realarm and a new user will be able to accept the notification
* The ability to manually complete (end) the notification by the user (if allowed by notification profile/category configuration)

## Notification List Views
* The high level state of each notification can be view from the list view
  * Active, accepted, inactive, undeliverable
* The notification icon animates when the ringtone for that specific notification is being played

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

## Ability to Select Notification Profile per Patient
* Scenarios where different patient profiles can have different notification configurations
* This opens up ability to assign patients based on teams. A notification profile can really be just a team.
  * We can put patients without a team to a default team or have some orchestrator determine which team should take the patient
  * This could depend on which team has least assigned patients per active members of the team

## Patient Dashboard
* View the assigned locations and the patients assigned to that location
* Overview of the total number of active notifications for each location
* Indicator if each notification is accepted or there are some pending notifications

## Prevent Closing of Application
* Android/iOS - Continues to function in background when logged in
* Linux/Windows - Prevents user from easily closing the app without confirming log off

## One App for Notification App and Configuration
* Depending on the roles available for the user logging in, they will either see the notification app, configuration app, or both.

## Web Version Info
* Only one tab can be active at a time
* Additional tabs will show an message indicating any other active tabs need to be closed before the new tab can function
* Once the other tabs are closed, the new tab will automatically refresh to the same URL that was requested
* WASM support

## Targeting Latest SDK versions
* Android 15 (SDK 35)

## Test Tool
* Ability to generate any category of notification
* Ability to generate a notification for any location within the system
* Ability to open the history for any generated notification
* Ability to track and end any generated notification
* Performance testing

# Other Notes
* Not a lot of unit tests for testing normal code.
* Dart is not great for data models serialization. Uses code generation. Example: Doesn't handle polymorphism for swagger/openapi definitions
* Mocking requires code generation
* No local(offline) demo mode (but we can instead host public test server for the sales folks)
* Multi-Screen support for Windows
* Have not deployed solution via Kubernetes yet