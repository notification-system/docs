# System Overview

## High Level Key Points
* Great developer experience
* Multi platform support
* Support for large enterprises
* Distributed architecture
* Supports most of the notification features of current Philips system plus additional concepts
* Working Architecture Solution: Backend, Frontend, Automation, Tooling

## Supported Platforms
* Android
* iOS (Work in progress on background notifications)
* Web
* Linux
* Windows

Ability to access native/unique features for each platform when necessary.

### Potential Support
* *MacOS* - Needs more testing. Likely requires additional entitlements/permissions.
* *Embedded Devices (Monitors)* - Potential to share some UI and packages across embedded devices.

## Developer Experience
### Backend
* Easy developer onboarding. Requirements:
  * Docker or Podman
  * Dotnet
  * IDE of choice

* Hot restart for individual services
* Open Telemetry Logs and Metrics
* Ability to easily stop/restart any service to test service recovery and system durability

### Frontend
* Extremely fast frontend UI development cycle with hot reload
* Develop the frontend on any desktop platform

## Enterprise Support
* Support for large customers
* 3000+ notifications per second
  * Disk: Gen3 NVMe, CPU: 7950x
  * Notifications contain simulated payload data (non-empty content)
  * Using a non-simple notification configuration (escalations, multiple users)
* Distributed architecture where each service recovers individually

## Automation
* Working automation pipeline
* Automation code coverage report for the entire system (backend and frontend)
  * Frontend Coverage: 45.6% (7227 out of 15856 actionable lines)
  * Backend Coverage: 65.8% (9472 out of 14386 coverable lines)
* End to end automation test cases for receiving notifications
* Ability to test any platform with same automation test cases

## Key Features
### Event Notification
#### Notification Profile
* Multiple notification profiles can be defined which determine the notification functionality.
* A notification profile can be assigned to a unit, facility, or the enterprise.
* This includes ability to delay notification, escalations levels, and notification settings.

##### Notifications Settings
Per notification category the following notification settings are possible:
* Behavior:
  * Notify - alerts users according to the configuration
  * Disabled - do not alert any user
  * Public Announcement - alerts all users
* Ringtone to play
* Priority (used for audio and visual priority)
* Minimum volume level (partial support in iOS)
* Turn on the screen (Android only)
* Unlock the device (Android only)
* Notification Visual:
  * Fullscreen - Opens directly to the user (Android/Windows only)
  * Local Notification 
* Vibrate (Android/iOS)
* Manual completion by accepting user  
  * Allows the user to mark notification as complete, removing it from their responsibility list
* Expiration

#### Custom Notifications
Code structured to add additional unique notification types with custom UI and data.
Currently support 4 unique data/visual types:
  1. Simple text notifications
  2. Patient alarms
  3. Lab report
  4. Undeliverable

#### Custom Notification Ringtones
* Ability to upload custom wav/mp3 sounds and configure the ringtones per category

#### Full Notification History
* Comprehensive tracking of all notification states and events
* Detailed delivery status for each user:
  - Sending → Delivered → Viewed → Accepted → Completed
* Other included statuses:
  - Escalation history (including reason)
  - Undeliverable (including reason)
  - Delayed
  - Expired
  - Reset
  - Disabled
  - Ended by the source system

#### Notification Matching
Default pattern for matching new notifications on any of the following:
* Location ID
* Patient ID
* Patient identifiers (MRN, SSN, Other ID)
* Location name (In progress)

Makes it easier to integrate notifications from third-party systems with varying available data.

### CDAS Support (within the current system)
The notification pipeline is always monitored. If the notification pipeline is broken, the user is notified about the disconnect with a visual of where the connection is broken.

### Security and Authentication
* All public APIs require access token
* Each device manages a refresh token and access token
* Server can invalidate refresh token based on configured limits (expiration, max active sessions per user)

### Distributed Patient Cache
* Simple ability to admit/update/discharge patients
* Any service can easily get a local cache of the currently assigned patients
* That readonly local cache is automatically synced with the source of truth

### Cross-Platform Deep Linking
* Uniform URLs work identically on Web/Android/iOS  
* Every screen automatically linkable

### Key Patterns
* **Automatic client synchronization** when backend data changes  
  ✓ Used by nearly all screens  
  ✓ Data refreshes automatically on reconnect or relevant service restart  
* **Shared resource repository**  
  ✓ Components needing the same data share a single repository instance  
  ✓ Data is loaded once from backend, kept updated by repository, and change notifications are broadcast to all components  
* **Graceful API error handling**  
  ✓ Client manages errors cleanly with user retry options  

### Long Term Storage
* All notifications are stored
* Report of total number of notifications grouped by category
* Fairly simple to code additional report types

## Platformization
* Possible modular design for the frontend and backend
* Any team can design and build their own UI package and backend service, then they can import it into the one solution

## Test Tool
* Ability to generate any category of notification
* Ability to generate a notification for any location within the system
* Ability to open the history for any generated notification
* Ability to track and end any generated notification
* Performance testing

## Public Test System
* Deployed system using 'Production' environment
* Publicly hosted over https

## Under Construction
* Full system performance testing
* iOS persistent background connection
* Unit tests
* Simulated/hardcoded elements:
  - Topology (64 rooms/unit, 20 units/facility, 20 facilities)
  - Users (20,000 sample set)
  - Permission
* Device management UI