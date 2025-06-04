# High Level Advantages
* Support for huge customers (26,000+ locations; 20,000+ users; 1000+ notifications per second)
* Improved developer experience for frontend and backend developers
  * No need to use/rely on cloud resources for development
* Support for non-mobile platforms
* Much faster development time for new features, entire system can be easily debugged
* Improved notification system workflow
* Architecture for working solution (backend, frontend, end to end automation, test tool)
* Custom notification data models without embedded images for much smaller footprint

## Single Repo Solution
* Cloud ready implementation that can work with Docker or Kubernetes
* Very simple to run/debug full system locally (developer friendly environment)
* Easy onboarding: Install an IDE, Dotnet and Docker or Podman
  * Can develop on Windows, Linux or MacOS
* No dealing with nested nuget packages and maintaining tens of repos.
  * But you can still leverage the microservices architecture and horizontal scaling
* Easily simulate service restart, service down scenarios
* If necessary, can still link to services from external repositories

## State Recovery on Restarts
* If any service crashes, on restart it will continue to function where it left off (unless there is a critical error)
* This includes the state of any previously active notifications
* There are 3 modes of state recovery on restart for the notification service
  * 1. We try to recover all previous active data and make sure we properly notify clients for all notifications, including notifications that occurred during downtime
    * Some clients will receive the same notification again, but the client gracefully handles this scenario and does not alarm again
  * 2. If that fails, then we recover starting from the last received notification before the crash (old state data is lost but no notification alarms are missed)
  * 3. No state recovery, we start processing only the new notifications as they come in
* The recovery mode depends on where we were in the process during the last run of the service. i.e. If we fail (restart/crash) during the operation of #1, then on next restart we try #2 and it keeps going down the chain.
* Once the service is running normally for 10 seconds, we reset the last recover state attempted and on future restart/crash we try to do a full state recovery.
* Smart recovery. Disabled, expired, ended, or completed notifications are not recovered and are minimally processed.
* Able to restart a system with 100k previously active notifications in a few seconds.
* Configurable recover time period. If the system is not restarted within the configurable period, then state recovery is bypassed.
  * This avoids scenarios where the system may be down for hours, everyone is well aware of a system outage, and the system does not try to recover state/data that occurred during downtime.
* In most scenarios, a crash of the system will not even be noticed by the users.

## Backend Storage
* EF Core using Postgres (Can migrate to other databases)
* Bulk storage solution for notification storage database
  * Safety check for duplicate key error when bulk storing, fallback to storing the updates individually and ignoring just the duplicates.
* Custom data models for unique notification types.
  * No images, meaning much smaller data transfers and less stress on the server to manage/generate images
  * Proper database structures so that you can perform more meaningful data mining for analytics
  * Easy process to add future custom notification types
* Retry mechanism for saving to the database for other exceptions
  * After certain amount of retries, the service is stopped and we rely on restart to continue
* Report of total number of notifications grouped by category
  * Fairly simple to code additional report types
* Easy to debug/view/modify storage data for the full system
* Data for cold storage is in a separate database which would make archiving easier

## Distributed Assigned Patient Cache
* Distributed patient cache for fast local access by services that need it frequently
* Eventual sync mechanism to make sure the distributed cache gets updated after any kind of connection issues
* This includes making sure the cache efficiently syncs after a potential crash during which the patient update is stored to the source db and but crashes before updating the distributed cache.
* Resetting the distributed cache if the patient data source of truth is reset.
  * Makes sure any existing connected readonly caches are efficiently reset if the distributed cache is reset.
* Making sure the entire patient data is sent to the distributed cache after a new instance of the distributed cache is created
* Resetting the distributed cache in scenarios where the pending updates to the cache are larger than the channel size

## Notification at Enterprise, Facility levels
* Depending on configuration, all notifications generated at a lower level can be handle at the parent location level
* The notification can be handled differently at each level, they each have their own escalation and notification settings
* Restoring notification state works for each level
* The main notification data is not duplicated, neither are status updates that affect the notification at all locations
* When viewing the notification from third party and the notification is handled at different levels, the user can view the status updates for each notification level individually.
* Possible to configure scenarios such as a Code Blue is broadcast immediately to the unit team. If the code blue is not addressed within X seconds, then it is broadcast to the facility level.

## CDAS
* Notification to the client when any part of the notification connection chain is broken.
* Each point of failure is separately indicated (MQTT, Notification Service, NATS, Unified Consumer)

## Notification Matching
Default pattern for matching any new notifications on any of the following:
* Location ID
* Patient ID
* Patient identifiers (MRN, SSN, Other ID)
* Location name

Makes it easier to integrate notifications from third-party systems with varying available data.
During matching, patient conflicts are detected. The resulting notification indicates a patient mismatch in such scenarios.

#### Custom Notifications
Code structured to add additional unique notification types with custom UI and data.
Currently support 4 unique data/visual types:
  1. Simple text notifications
  2. Patient alarms
  3. Lab report
  4. Undeliverable

All code paths that need to be modified to support a new notification type are marked with: //* NEW_NOTIFICATION_TYPE_EDIT

##### Undeliverable Notification
* Configuration to determine how many seconds a client has to respond to a notification before it is considered undeliverable.
* The system generates an Undeliverable notification if a notification cannot be delivered to any end device. Exceptions to this include:
  * The notification category is disabled by configuration
  * The undeliverable notification is of type undeliverable (prevents undeliverable loop)
* This undeliverable notification can be fully configured like any other notification category

## Security
* Refresh token management. Refresh tokens are recycled after each use.
* Only internal systems can publish notifications to the message bus
  * Internal services use private keys shared via environment variables
* All public APIs are protected
  * Valid JWT is required for all HTTP requests and MQTT connections
* Production system requires HTTPS
  * TLS or secure web sockets are required for MQTT

## User Session Control
* Configurable max user session period. This is the validity period of the refresh token managed by the backend.
* Configurable max number of sessions a single user can have. The oldest session will be kicked off if the number surpasses the configured limit.
  * The device with the ended session is immediately logged off if it is online with a message to the user.
  * By enabling multiple sessions, this allows the user to maintain their login on mobile but still connected to a desktop or web version at the same time.
* All user data/state is maintained by the backend. This means the user can seamlessly log in on a different device without losing any data/configuration.
  * Only the UI Theme configuration can be unique per device.

# Other Implementation Details
* Null Safety enabled across solution
* Targets .NET 9
* Immediately escalate the notification if there are no assigned users or no assigned roles
* Configurable global TTL strategy for notifications (can be overridden for each category within a notification profile)
* EF Core migration strategy
* Basic API versioning strategy to force older clients to update.
* Tree based notification categories
* Open telemetry metrics for monitoring notification service. Including number of active notifications, number of pending updates, authenticated users, stored data count, channel size, pending batched saves ...etc.
  * Some telemetry data is collected only when in Development environment
* Each service capable of gracefully handling network issues or other service restarts.
  * i.e. If history service crashes it will continue from where it left off on restart. MQTT unreachable, messages will queue until MQTT is available again.
* Notification service maintains only simpler notification data models. Larger custom data is only available in the history service.
  * This allows for hundreds of thousands or even millions of active notifications at a time.
* Proper caching for almost all data that is read often
* Proper CORS policy for the current 'Production' server

# System Debugging
* Easy to use development tools available via Aspire
* Ability to view all traces across the entire system
* View, filter logs for all or individual services
* Monitor metrics for performance investigation across the system
* Dashboard to monitor the health of the system and indicate any errors that may have occurred
* View environment variables, references, resources of any individual service
* Access and test any Rest API for any service from the dashboard
* Web based frontend app and test tool embedded into the backend service

## Other Info
* None of the services communicate with each other directly (Call Rest API on another service)
  * Avoided all designs that required that since it complicates the system
  * Can easily plug this system into a solution that has its own Topology Management, User Management and Patient Management.
* Optimized performance without micromanagement. However, the following optimizations could further enhance efficiency:
  * MemoryPack Serialization – For faster serialization/deserialization (currently using JSON).
  * Minified Subscription Topics – To streamline message routing and reduce overhead.
  * Spans – Reduce memory allocations and enable high-performance looping over buffers.
* Code has good comments for non-trivial code paths
* DTO, Database, Domain models separation
* Source of truth pattern for the data sources within the service
* All APIs that require finding patient, location, notification, user, assignment, ringtone, or any unbounded list are accessed with some form of hash map