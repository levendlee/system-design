# Chapter 10: Design A Notification System

(Not an interesting topic)

## Step 1 - Understand the problem and establish design scope

- Type
    - Push notification/SMS/Email
- Real-time
    - Soft. Dely acceptable.
- Supported devices
    - iOS. Android. PCs.
- What triggers
    - Triggered on client. Scheduled on server.
- Opt out?
    - Yes.
- How many
    - 10M mobile. 1M SMS. 5M emails.

## Step 2 - Propose high-level design and get buy-in
### Different types of notifications
- Provider
    - Identifier: Device token/Phone number/Email address
    - Payload: JSON
- Service
    - iOS:      APNS (Apple Push Notification Service)
    - Android:  FCM (Firebase Cloud Messaging)
    - SMS:      SMS Service (Twilio, Nexmo, etc)
    - Email:    Commercial email services (Sendgrid, Mailchimp, etc)
- Client
    - User: iOS/Andriod/Mobile/Email

### Contact info gathering flow
User -> (Sign Up) -> Load Balancer -> API servers -> (Store contact info) -> Database

Database: User table and device table.

### Notification sending/recieving
#### High level design
- Service (1~N)
    - Mirco-service. Cron job. Distributed system.
- Notification System
    - Centerpiece.
- Third party services (APNS/FCM/SMS/Email)
    - Pay attention to extensibility.
- Endpoints(iOS/Android/SMS/Email)

Problems
- Single point of failure (SPOF): *Notification System*.
- Hard to scale: DB, cache, notification processing components.
- Performance bottleneck: Resource intensivce.

#### HIgh level design (improved)
##### Improvements
- ***Move database and cache out of the notification server***.
- ***Add more notification servers and automatic horizontal scaling***.
- ***Introduce message queues to decouple the system components***.

##### Notification servers
- Internal **APIs** to send notifications. RESTFul.
- Basic **validations** on emails, phone numbers, etc.
- Queury **database or cache** to fetch data needed to render a notifcaiton.
- Put notification data to **message queues** for parallel processing.

##### Cache
User info. Device info. Notification templates.

##### DB
Data about user, notification, settings.

##### Message queues
Decouple and serve as buffers.

##### Workers
Pull notification events from queue and send to third-party services.

## Step 3 - Design deep dive
### Reliability
#### How to prevent data loss
Notification system persists notification data in a DB and implements retry.

#### Will recipients receive a notification exactly once
No. Dedupe. When notification first arrives, check if seen before and discard if so.

### Additional components and considerations

#### Notification template

#### Notification setting
Fine-grain control. Notification setting table. e.g. id/channel/opt_in.

#### Rate limiting
Limit the number of notifications a user can receive.

#### Retry mechanism
Third party failed. Add back to message queue. Alert if persists.

#### Security in push notifications
iOS-appKey. Android-appSecret.

#### Monitor queued notifications
Queue is large -> add more workers.

#### Events tracking - Analytics Service
Open rate/click rate/engagement. 

## Step 4 - Wrap Up
- Reliability
- Security
- Tracking and monitoring
- Respect user settings
- Rate limiting
