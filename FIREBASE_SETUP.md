# Firebase Setup Instructions

This document explains how to configure Firebase for the Coin of Fate multiplayer and global statistics features.

## Features Requiring Firebase

1. **Global Doom Barometer** - Real-time global flip statistics
2. **Séance Mode** - Multiplayer room system for synchronized flips

## Quick Setup (5 minutes)

### Step 1: Create a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click "Add project" or "Create a project"
3. Enter project name: `coinoffate` (or your preferred name)
4. Disable Google Analytics (optional, not needed for this project)
5. Click "Create project"

### Step 2: Enable Realtime Database

1. In your Firebase project, click on "Realtime Database" in the left menu
2. Click "Create Database"
3. Choose a location (closest to your users)
4. Start in **test mode** (we'll secure it later)
5. Click "Enable"

### Step 3: Get Your Firebase Configuration

1. Click the gear icon ⚙️ next to "Project Overview"
2. Select "Project settings"
3. Scroll down to "Your apps"
4. Click the web icon `</>` to add a web app
5. Register app name: "Coin of Fate Web"
6. Copy the `firebaseConfig` object

### Step 4: Update index.html

Open `index.html` and find this section (around line 624):

```javascript
const firebaseConfig = {
    apiKey: "AIzaSyDEMO-REPLACE-WITH-YOUR-KEY",
    authDomain: "coinoffate-demo.firebaseapp.com",
    databaseURL: "https://coinoffate-demo-default-rtdb.firebaseio.com",
    projectId: "coinoffate-demo",
    storageBucket: "coinoffate-demo.appspot.com",
    messagingSenderId: "123456789",
    appId: "1:123456789:web:abcdef123456"
};
```

Replace it with your actual Firebase config from Step 3.

### Step 5: Configure Database Rules

1. In Firebase Console, go to "Realtime Database"
2. Click on the "Rules" tab
3. Replace the rules with:

```json
{
  "rules": {
    "globalStats": {
      ".read": true,
      ".write": true
    },
    "rooms": {
      ".read": true,
      "$roomCode": {
        ".write": true,
        ".indexOn": ["createdAt"]
      }
    }
  }
}
```

4. Click "Publish"

**Note:** These rules allow anyone to read/write. For production, you should add authentication.

## Testing

1. Open your website
2. Click "ENTER THE MAGIC"
3. The Global Doom Barometer should show "0 Total Flips" instead of "Firebase not configured"
4. Try creating a Séance room - it should generate a 6-character room code
5. Open a second browser window and join the room using the code
6. When the host flips the coin, both windows should show the same result

## Database Structure

Your Firebase Realtime Database will have this structure:

```
{
  "globalStats": {
    "heads": 0,
    "tails": 0
  },
  "rooms": {
    "ABC123": {
      "host": "user_abc123",
      "createdAt": 1700000000000,
      "participants": {
        "user_abc123": {
          "name": "Host",
          "joinedAt": 1700000000000
        },
        "user_def456": {
          "name": "Guest 42",
          "joinedAt": 1700000100000
        }
      },
      "lastFlip": {
        "result": "H",
        "timestamp": 1700000200000
      }
    }
  }
}
```

## Security (Production)

For production use, update your database rules to:

```json
{
  "rules": {
    "globalStats": {
      ".read": true,
      ".write": "auth != null"
    },
    "rooms": {
      ".read": "auth != null",
      "$roomCode": {
        ".write": "auth != null",
        ".indexOn": ["createdAt"],
        ".validate": "newData.hasChildren(['host', 'createdAt', 'participants'])"
      }
    }
  }
}
```

Then implement Firebase Authentication (optional).

## Cleanup Old Rooms (Optional)

Rooms don't automatically expire. To clean up old rooms:

1. Go to Firebase Console → Realtime Database
2. Navigate to `rooms/`
3. Manually delete old room codes

Or set up a Cloud Function to auto-delete rooms older than 24 hours:

```javascript
// Cloud Function (requires Firebase Blaze plan)
exports.cleanupOldRooms = functions.pubsub
  .schedule('every 24 hours')
  .onRun(async (context) => {
    const cutoff = Date.now() - (24 * 60 * 60 * 1000);
    const snapshot = await admin.database().ref('rooms')
      .orderByChild('createdAt')
      .endAt(cutoff)
      .once('value');

    const updates = {};
    snapshot.forEach(child => {
      updates[child.key] = null;
    });

    return admin.database().ref('rooms').update(updates);
  });
```

## Troubleshooting

### "Firebase not configured" error

- Check that you replaced the demo config with your actual Firebase config
- Verify the Firebase SDK scripts are loading (check browser console)
- Make sure your database URL is correct

### Multiplayer not syncing

- Check Firebase Console → Realtime Database → Data tab
- Verify the room exists under `rooms/`
- Check browser console for errors
- Ensure database rules allow read/write access

### Global stats not updating

- Check Firebase Console → Realtime Database → Data
- Look for `globalStats/heads` and `globalStats/tails`
- If missing, manually create them with value `0`
- Check database rules allow writing to `globalStats`

## Cost Estimate

Firebase Free Plan (Spark):
- Realtime Database: 1 GB stored, 10 GB/month downloaded
- Should handle 1000-5000 daily users easily
- **Cost: $0/month**

If you exceed free tier:
- Pay as you go (Blaze plan)
- Estimated cost for 10,000 daily users: ~$5-10/month

## Support

If you need help:
1. Check Firebase Console for error messages
2. Open browser DevTools Console (F12) for JavaScript errors
3. Review [Firebase Realtime Database documentation](https://firebase.google.com/docs/database)

---

**Last updated:** 2025-11-18
**Firebase SDK Version:** 10.7.1
**Required Firebase Services:** Realtime Database only
