# High Level Advantages
* Support for huge customers (26,000+ locations; 20,000+ users; 3000+ notifications per second)
* Improved developer experience for frontend and backend developers
  * No need to use/rely on cloud resource for most development
* Support for non-mobile platforms
* Much faster development time for new features, especially related to UI
* Improved notification system workflow
* Architecture for working solution (backend, frontend, end to end automation)
* Not constrained by fitting all notifiction types into a single model

# Implemented Features
* Null Safety implemented by default
* Effecient message bus from the unified consumer to the notification service and persistant storage
* Can easily consume new type of notification and convert to one of the several existing formats.
  * Or can create a special/customizable notification that the system can support
* Immediately escalate the notification if there are no assigned users
* Database migration strategy
* Categories tree structure
* Open telemetry metrics for monitoring notification service. Including number of active notifications, number of pending updates, authenticated users, stored data count, channel size ...etc.
  * Some telemetry data is collected only when in Development environment
* Each service capable of gracefully handling network issues or other service restarts.
  * i.e. If history service crashes it will continue from where it left off on restart. MQTT unreachable, messages will queue until MQTT is available again
* Custom data for notification updates
* Notification service maintains only simpler notification data models. Larger custom data is only available in the history service.
  * This allows for hundred of thousands or even millions of active notification at a time.
* Configurable max user session period. This is the validity period of the refresh token managed by the backend.
* Configurable max number of sessions a single user can have. The oldest session will be kick off if the number surpasses the configuration.
  * The end device is immediately logged off if it is online with a message to the user.
* Undeliverable notification state when the notification cannot be delivered to end devices
  * Undeliverable scenarios: no configuration, no assigned roles, no assigned users, the users are offline, or the notification delivery was not confirmed by any device after reaching the end of escalation.
  * This also generated a configurable undeliverable notification
* The location active notifications list is automatically updated
* Proper caching for almost all data that is read often
* Proper CORS policy for current 'Production' server
* All APIs that require finding patient, location, notification, user, assignment, ringtone, or any non bounded list are accessed with some form of Hashmap.

## Distributed Patient Cache
* Distributed patient cache for fast local access by services that need it frequently
* Eventual sync mechanism to make sure the distribute cache gets updated after any kind of connection issues
* This includes making sure the cache effeciently syncs after a potential crash during which the patient update is stored to the source db and but crashes before updating the distribute cache.
* Reseting the NATS distributed cache if the patient data source of truth is reset.
* Handle scenario if a receiving service is already connected. Those already active services will be sent a signal to reset the cache.
* Making sure the entire patient data is sent to NATS after new instance of NATS is used
* Reseting the distributed cache in scenario where our pending updates to the cache are larger than the channel size

## State Recovery on Restarts
* If any services crashes, on restart it will continue to function where it left off (unless there is a critical error)
* This includes the state of any previously active notifications
* There are 3 modes of state recovery on restart for the notification service
  * 1. We try to recover all previous active data and make sure we properly notified clients for all notifications, including notifications that occurred during downtime
    * Some clients will receive the same notification again, but the client gracefullly handles this scenario and does not alarm again
  * 2. If that fails, we try to make sure any in between notifications are recovered (old state data is lost but no notification alarms are missed)
  * 3. No state recovery, we start processing only the new notifications as they come in
* The recovery mode depends on where we were in the process during the last run of the service. i.e. If we fail (restart/crash) during the operation of #1, then on next restart we try #2 and it keeps going down the chain.
* Once the service is running normally for 10 seconds, we reset the last recover state attempted and on future restart/crash we try to do a full state recovery.
* Smart recovery, where disabled, expired, ended or completed notifications are not recovered.
* Able to restart a system with 100k previously active notifications in a few seconds.

## Single Repo Solution
* Cloud ready implementation that can work with Docker or Kubernetes
* Very simple to run/debug locally (very developer friendly environment)
* Download Visual Studio and Docker and the solution will run out of the box
* No dealing with nested nuget packages and maintaining tens of repos.
  * But you can still leverage the microservices architecture and horizontal scaling

## Backend Storage
* EF Core using Postgres (Can migrate to other databases)
* Bulk storage solution for notification storage database
  * Safety check for duplicate key error when bulk storing, fallback to storing the updates individually and ignoring just the duplicates.
* Custom data models for unique notification types.
  * No images, meaning much smaller data transfers and less stress on the server to manage/generate images
  * Proper database structures so that you can perform more meainingful data mininig for analytics
  * Easy process to add future custom notification types
* Retry mechanism for saving to the database for other exceptions
  * After certain amount of retries, the service is stopped and we rely on restart to continue

## Notification at Enterprise, Facility levels
* Depending on configuration, all notifications generated at lower level can be handle at the parent location level
* The notification can be handled differently at each level, they each have their own escalation and notificaton settings
* Restoring notification state works for each level
* The main notification data is not duplicated, neither are status updates that affect the notification at all locations
* When viewing the notification from third party and the notification is handled at different levels, the user can view the status updates for each notification level individually.
* Possible to configure scenarios such as a Code Blue is broadcast immediately to the unit team. If the code blue is not addressed within X seconds, then it is broadcast to the facility level.

## CDAS
* Notification to the client when any part of the notification connection chain is broken.
* Each point of failure is seperately indicated (MQTT, Notification Service, NATS, Unified Consumer)

## Notification Matching
Default pattern for matching new notifications on any of the following:
* Location ID
* Patient ID
* Patient identifiers (MRN, SSN, Other ID)
* Location name (In progress)

Makes it easier to integrate notifications from third-party systems with varying available data.
During matching, patient conflicts are detected. The resulting notification indicates a patient mismatch in such scenarios.

# Projects
## System.AppHost
The main starting point to run a local instance of the system. This is built on top of .NET Aspire to help manage local environment.

## TODO
* Aliasing for locations
* Audit Log for all changes
* DateTime source of truth? If new event has older timestamp, then should it auto escalate or the timestamp assumed to be new relative to the notification system?
  * Initial thought: If the timestamp is older than current time, then assume the timestamp is correct and maybe there was a delay in receiving the notification. If the gap is large, then log it as warning.
  * If the timestamp is newer than current time, then assume the timestamp is incorrect. Process it as brand new notification but log an error. Maybe even a specific notification category for this error.

## Other Info
* None of the services communicate with each other directly (Call Rest API on another service)
  * Avoided all designs that required that since it complicates the system
* Optimized performance without micromanagement. However, the following optimizations could further enhance efficiency:
  * MemoryPack Serialization – For faster serialization/deserialization.
  * Minified Subscription Topics – To streamline message routing and reduce overhead.
  * Spans – Reduce memory allocations and enable high-performance looping over buffers.
* Code has good comments for non-trivial code paths
* DTO, Database, Domain models separation
* Source of truth pattern for the components within the service