# Turning on one-tap Google sign-in

**Who this is for:** you, once. After this, anyone you send the app to just taps
*Sign in with Google* and starts using it. No Firebase, no config, no setup on their side.

**Time:** about 10 minutes. **Cost:** free. You do not need to write any code — you paste
one block of text into one line of one file.

**Nothing breaks while you do this.** Until you finish, the app keeps working exactly as it
does today, including on your phone and tablet.

---

## What you are actually building

One Firebase project that the app talks to. Inside it, every person who signs in gets their
own private drawer, keyed to their Google account.

```
your project
├── ledgers/<your Google id>          ← your phone + tablet
├── ledgers/<girlfriend's Google id>  ← her Pixel, invisible to you
└── ledgers/<a friend's Google id>    ← invisible to both of you
```

The security rules make that separation real: the *server* refuses to hand one person another
person's drawer. It is not a UI trick. You will paste those rules in step 5.

You will be the project owner, so you can see usage and shut it down — but the data itself
is per-account.

---

## Step 1 — Create the project

1. Go to **console.firebase.google.com** and sign in with your Google account.
2. Click **Create a project**.
3. Name it something like `mistake-ledger`. Click **Continue**.
4. Google Analytics: **turn it off** (you do not need it). Click **Create project**.
5. Wait for it to finish, then click **Continue**.

## Step 2 — Add a web app and copy the config

1. On the project home page, click the **`</>`** icon (it says *Web*).
2. Nickname it `ledger`. Do **not** tick "Firebase Hosting". Click **Register app**.
3. You will see a code block containing `const firebaseConfig = { ... }`.
4. **Copy just the part inside the curly braces, including the braces.** It looks like:

```
{
  apiKey: "AIzaSyC...",
  authDomain: "mistake-ledger.firebaseapp.com",
  projectId: "mistake-ledger",
  storageBucket: "mistake-ledger.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
}
```

Paste it somewhere safe for a minute (Notes, a message to yourself).

> This block is **not** a password. It is a public address label — it ships inside every web
> app that uses Firebase. What protects your data is the sign-in plus the rules in step 5.

## Step 3 — Turn on Google sign-in

1. Left sidebar → **Build → Authentication** → **Get started**.
2. Choose **Google** from the provider list.
3. Toggle **Enable**.
4. Pick your own email as the *project support email*.
5. Click **Save**.

## Step 4 — Create the database

1. Left sidebar → **Build → Firestore Database** → **Create database**.
2. Choose **Production mode**. Click **Next**.
3. Pick the region closest to you (for India: `asia-south1`). Click **Enable**.
4. Wait for it to finish.

## Step 5 — Paste the security rules *(this is the important one)*

1. Still in Firestore Database, open the **Rules** tab at the top.
2. Delete everything in the box and paste exactly this:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{db}/documents {
    match /ledgers/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

3. Click **Publish**.

This one rule is what keeps everyone's data separate. It says: you may open the drawer named
after your account, and no other. Do not skip it, and do not leave the database in test mode.

## Step 6 — Allow your website

1. Go back to **Authentication → Settings** tab → **Authorized domains**.
2. Click **Add domain** and add: `yoyo196204-hub.github.io`
3. `localhost` is usually already there. Leave it.

If you ever move the app to another address, add that one here too, or sign-in will fail with
an "unauthorized domain" message.

## Step 7 — Put the config into the app

1. Open `index.html`.
2. Search for `APP_FB_CFG`. You will find one line that reads:

```js
const APP_FB_CFG = null; /* ← paste your firebaseConfig object here, e.g. ... */
```

3. Replace `null` with the block you copied in step 2, so it becomes:

```js
const APP_FB_CFG = {
  apiKey: "AIzaSyC...",
  authDomain: "mistake-ledger.firebaseapp.com",
  projectId: "mistake-ledger",
  storageBucket: "mistake-ledger.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

Keep the `const APP_FB_CFG =` at the start and the `;` at the end.

4. Save the file, commit, and push. Once GitHub Pages updates, the **Data** screen shows
   **Sign in with Google** for anyone who opens the app — with nothing to set up.

---

## Moving your own two devices across

Right now your Nord CE 5 and your Pad 3 sync through the Firebase project you set up earlier.
They will keep doing that and will **not** be moved automatically — that is deliberate, so
nothing changes under you.

When you are ready to move them to the shared project, do this **one device at a time**:

1. On the phone: **Data → Save / share a backup** and keep the file somewhere safe. (Belt and
   braces — the app also snapshots automatically in the next step.)
2. **Data → Advanced — use my own project → Remove.**
   The app snapshots your data, switches to the shared project, and asks you to sign in.
3. Tap **Sign in with Google** and choose your usual account.
4. Your data uploads into your drawer in the new project. Check the Data screen: it should
   show your email and a sensible "in cloud" count.
5. Only once the phone looks right, repeat on the tablet. When the tablet signs in, its data
   merges with what the phone uploaded — nothing is replaced.

Your old project keeps its copy until you delete it. Leave it alone for a week as a safety net.

## Your girlfriend's Pixel

Her data must stay hers, and it will:

1. She opens the app and taps **Sign in with Google** with **her own** account.
2. Her existing data on that phone uploads into **her** drawer.
3. You cannot see it, and she cannot see yours — different accounts, different drawers, enforced
   by the rules from step 5.

**The one thing to avoid:** signing into *your* account on *her* phone (or hers on yours). If
that happens the app will not silently combine anything — it stops and shows a *"Whose ledger
is this?"* screen with two clear choices, and it snapshots first either way. Choose
**Keep them separate**.

## Sending it to friends

Just send the link. They open it, tap **Sign in with Google**, and they are done. Their data
is theirs.

If someone wants their data in a project they control instead, the **Advanced** option is still
there and still works exactly as before.

---

## If something goes wrong

| What you see | What it means | Fix |
|---|---|---|
| "This site isn't allow-listed" | The domain is missing | Step 6 — add `yoyo196204-hub.github.io` |
| "Turn on Google sign-in" | Provider not enabled | Step 3 |
| "The cloud database is blocking access" | Rules missing or wrong | Step 5 — paste and **Publish** |
| "No Firestore database found" | Database not created | Step 4 |
| Sign-in popup does nothing on a phone | Normal for installed apps | The app already falls back to a full-page redirect; just tap again |
| "Whose ledger is this?" | Two accounts on one device | Pick **Keep them separate** unless both are genuinely you |

**Nothing is ever deleted without a snapshot first.** Data → *Auto-backups on this device*
holds the last few, and restoring merges rather than overwrites.

## Keeping an eye on cost

The free tier is 50,000 reads and 20,000 writes a day. A person triaging a test writes a
handful of times. A few dozen friends will not come close. In Firebase, **Usage and billing**
shows the numbers, and with no card on file it can never charge you — it just stops.
