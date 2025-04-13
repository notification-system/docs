# High Level Advantages
* Immediate support for huge customers (26,000+ locations; 20,000+ users; 3000+ notifications per second)
* Improved developer experience for frontend and backend developers
  * No need to use/rely on cloud resource for most development
* Support for non-mobile platforms
* Much faster development time for new features, especially related to UI
* Improved notification system workflow
* Architecture for working solution (backend, frontend, end to end automation)
* Knowledge transfer to new developers
* Not constrained by fitting all notifiction types into a single model

# Implemented Features
* Null Safety implemented by default
* Effecient message bus from the unified consumer to the notification service and persistant storage
* Can easily consume new type of notification and convert to one of the several existing formats.
  * Or can create a special/customizable notification that the system can support
* Immediately escalate the notification if there are no subscribers to the notification
* Database migration stategy
* Categories tree structure and settings manager
* Open telemetry metrics for monitoring notification service. Including number of active notifications, number of pending updates, authenticated users, stored data count, channel size ...etc.
  * Some telemetry data is collected only when in Development environment
* Safe history database saves. Check for duplicate key exceptions which could occur in rare scenario when crashing after database save but before acknowledgement to NATS.
* If the Notification History service crashes, it will continue to save data where it left off after a restart
* If connection to NATS is lost, the services will recover after NATS is available again.
* Undeliverable notifications
* Custom data for notification updates
* Notification service maintains only simpler notification data models. Larger custom data is not cached.
* Disabled notifications are not recovered but do show the disabled status in the history.
* Configurable max user session period. This is the validity period of the refresh token managed by the backend.
* Configurable max number of sessions a single user can have. The oldest session will be kick off if the number surpasses the configuration.
  * The end device is immediately logged off if it is online with a message to the user.
* Undeliverable notification state when the notification cannot be delivered to end devices
  * Undeliverable scenarios: no configuration, no assigned roles, no assigned users, the users are offline, or the notification delivery was not confirmed by any device after reaching the end of escalation.
* Matching notification by location id, patent internal id, patient identifiers (mrn, ssn, other id)
  * TODO : Location name
* The location active notifications list is automatically updated
* Proper caching for almost all data that is read often
* Proper CORS policy for current 'Production' server

## Distributed Patient Cache
* Distributed patient cache for fast local access by services that need it frequently
* Eventual sync mechanism to make sure the distribute cache gets updated after any kind of connection issues
* This includes making sure the cache effeciently syncs after a potential crash during which the patient update is stored to the source db and but crashes before updating the distribute cache.
* Reseting the NATS distributed cache if the patient data source of truth is reset.
* Handle scenario if a receiving service is already connected. They will be sent a signal that cache will be reset.
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
* The recovery mode depends on where we were in the process during the last run of the service. i.e. If we fail (restart/crash) during #1, then on next restart we try #2 and it keeps going down the chain.
* Once the service is running normally for 10 seconds, we reset the last recover state attempted and on future restart/crash we try to do a full state recovery.

## Single Repo Solution
* Cloud ready implementation that can work with Docker and Kubernetes
* Very simple to run/debug locally (very developer friendly environment)
* Download Visual Studio and Docker and the solution will run out of the box
* No dealing with nested nuget packages and maintaining tens of repos.
  * But you can still leverage the microservices architecture and horizontal scaling

## Backend Storage
* EF Core using Postgres (Can migrate to other databases)
* Bulk storage solution for notification storage database
  * Safety check for duplicate key error when bulk storing, fallback to storing the updates individually and ignoring just the duplicates.
* Custom data models for unique notification types.
  * No images, meaning much smaller data transfers and less stress on the server to generate images
  * Proper database structures so that you can perform more meainingful data mininig for analytics
  * Easy process to add future custom notification types
* Retry mechanism for saving to the database for other exceptions
  * After certain amount of retries, the service is stopped and we rely on restart to continue

## Notification at Enterprise, Facility levels
* Depending on configuration, all notifications generated at lower level can be handle at the parent location level
* The notification can be handled differently at each level, they each have their own escalation and notificaton settings
* Restoring state still work correctly for each level
* The main notification data is not duplicated, neither are status updates that affect the notification at all locations
* When viewing the notification from third party and the notification is handled at different levels, the user can view the status updates for each notification level individually.
* Possible to configure scenarios such as a Code Blue is broadcast immediately to the unit team. If the code blue is not addressed within X seconds, then it is broadcast to the facility level.

## CDAS
* Notification to the client when any part of the notification connection chain is broken.
* Each point of failure is seperately indicated (MQTT, Notification Service, NATS, Unified Consumer)

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