# Getting Started

Below is Background Location and Remote Config System Design for TMS Driver App 1.1.0

![Architecture](/architectural_overview.png?raw=true "created by figjam")

# Requirements and Goals of the System.

## Functional Requirements:

1. Given their location (longitude/latitude), drivers should be able to update location.
2. Drivers should be able to be notified when new version of the app released.

## Non-functional Requirements: 

1. Drivers should have real-time location tracking experience with minimum latency.
2. Our service should support heavy work load. There will be a lot of update requests.

# Background Location in Flutter.

<YouTube id="b0I1Xq_iSK4" />

# Firebase Remote Config with Google Analytic

[firebase](https://firebase.google.com/docs/remote-config/config-analytics) states "When you build an app that includes both Firebase Remote Config and Google Analytics, you gain the ability to understand your app users better and to respond to their needs more quickly. You can use Analytics audiences and user properties to customize your app for segments of your user base with flexibility and precision."

## Benefit.

1. Make updates without republishing.
2. gives you visibility and fine-grained control over your app's behavior and appearance so you can make changes by simply updating its configuration from the Firebase console.
3. Confidently/Progressively roll out new features. feature flags so you can gradually roll out new features to ensure they are stable and performing well, before rolling them out broadly. If the new features don't meet expectations or cause crashes, you can quickly roll them back.
4. Personalize your app for different audiences

## Data Structure

```json
{
"conditions": [
      {
        "name": "internalUser",
        "expression": "audience == 'internalUser'"
      }
    ],
    "version": {
        "versionString": "1.0.4",
        "minimumRequiredVersion": "1.0.0",
        "updateTime": 1653883939,
        "updateType": "INCREMENTAL_UPDATE"
    },
    "personalizations": {
        // generics 
    },
    "featureFlags": {
        "backgroundLocation": {
            "defaultValue": {
                "value": false
            },
            "conditionalValues": {
                "internalUser": {
                    "value": true
                }
            }
        }
    }
}
```

# Cloud Functions V2

What's new in Cloud Functions for Firebase v2
- Function instances can now execute more than one request at a time.
- Secure your callable and HTTP functions.
- Cloud Functions is now built on [Cloud Run](https://cloud.google.com/run)
- Function instances now default to the default compute service account rather than the app engine service account.
- HTTP functions can now have a 1 hour timeout (up from 9 minutes previously).
- The `firebase-functions` SDK has been reimagined as more native to modern JavaScript
- [more](https://firebase.google.com/docs/functions/beta)

Specification

- region: `us-central1` (multi region).
- frameworks: express.
- minimum instance: 0.
- functions: one https function, one Firebase remote config trigger and one auth user trigger.
- scheduler: cron job `'1 of month 00:00'`
- API Endpoints for https function: Get Last location and Update Location.
- Environments: Development and Production.

Code Snippet

```typescript
exports.scheduledFunction = functions.pubsub.schedule('1 of month 00:00')
  .timeZone('America/New_York')
  .onRun((context) => {
    // TODO(Ridwan): Log event.
    // TODO(Ridwan): Check every user locations collection. delete collection `if data > 30 days` 
                     and remove Document if no any activities under the last 30 days
});

```

# Firestore


## Data Structure

for each document will be located at `/user.location.trackings/${uid}/${MMMM yyyy}/${documentId}`.<br>
for example `/user.location.trackings/jFDcLYLoP8QB1kltN6jctC5UYSD3/january 2022/pSUZ2WXyP0YuItzThc15`

<img width="400" alt="image" src="https://user-images.githubusercontent.com/56130955/170919462-73353071-4401-4818-a571-1c798a7d09d7.png">
<img width="400" alt="image" src="https://user-images.githubusercontent.com/56130955/170919559-cc0c98ff-59d5-4e8f-9b8d-b56dd1a8f234.png">


there will one be collections with name `user.location.trackings` and subcollections with name `${MMMM yyyy}`.
Firebase Firestore Document holds information about `createdTime` and `updateTime` so we dont need them in document data.

```json
{
    "lat":89.11111111,
    "long": 179.11111111
}
```
