# Letting friends sign in with Google

## If you already use sync (that's you) — this is the whole thing

You already have everything set up. You do **not** need to make a new project or touch the
Firebase website. There is exactly **one** thing left:

**1.** On your phone or tablet, open the app → **Data** tab → **Sync settings** → tap
**Copy my config**.

**2.** Send me what it copied. I'll put it in and deploy it.

That's it. Once it's live, anyone you send the link to just taps *Sign in with Google* and
starts using the app. Your data and theirs stay completely separate.

> **Is it OK to share that config?** Yes. It's an address label, not a password — every
> website using Firebase has one visible in its page source. Your data is protected by the
> Google sign-in and the rules already on your database, not by keeping this secret.

If you'd rather paste it in yourself: open `index.html`, find the line that says
`const APP_FB_CFG = null;`, and replace the word `null` with what you copied. Keep the `;` at
the end.

**Nothing changes for you.** Your phone and tablet keep syncing exactly as they do now.

---

## What your friends will do

Send them the link. They open it, tap **Sign in with Google**, done. No setup on their side.

## What about your girlfriend's Pixel?

Nothing to do. She taps **Sign in with Google** with **her own** Google account and her data
goes into her own space. You can't see hers; she can't see yours. Different Google account,
different data — the database enforces that, it isn't just hidden on screen.

**The one thing to avoid:** signing into *your* Google account on *her* phone, or hers on
yours. If that ever happens by accident, the app won't quietly mix anything — it stops and
shows a screen called *"Whose ledger is this?"* with two buttons. Pick **Keep them separate**.
It saves a backup either way.

---

## One rules update needed for Rivals

The **Rivals** tab (comparing your numbers with a study partner's) needs one extra rule, because
a scoreboard has to let group members read each other's rows. Your **ledger stays private** —
this only opens the small scoreboard rows, which contain no mistakes.

Go to **console.firebase.google.com → logger-b9d1a → Build → Firestore Database → Rules**,
replace everything with this, and click **Publish**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{db}/documents {

    // your ledger — only you, exactly as before
    match /ledgers/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }

    // Rivals scoreboard — signed-in people who know the group code can read the rows,
    // but you can only ever write your own
    match /groups/{gid}/members/{uid} {
      allow read: if request.auth != null;
      allow write, delete: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

Until you publish this, the Rivals tab will say the database is blocking access. Everything else
keeps working normally.

> Group codes are random 8-character strings. Someone would have to be signed in **and** know
> your exact code to see the board. If that ever bothers you, leave the group — your row is
> deleted immediately.

## How the separation actually works (skip if you don't care)

Everyone shares one database, but each person gets their own locked drawer:

```
the project
├── your Google account        →  your mistakes, reads, plan
├── her Google account         →  hers, invisible to you
└── each friend's account      →  theirs, invisible to both of you
```

The lock is one rule already saved on your database: *you may open the drawer with your own
name on it, and no other*. The server refuses everything else.

---

## Later: moving your two devices onto the shared setup

**You don't need to do this.** Your Nord and Pad already sync fine and will keep working.
Only do this if you want everyone — including you — on one single setup.

Do it **one device at a time**, phone first:

1. **Data → Save / share a backup**, keep the file somewhere safe.
2. **Data → Sync settings → Remove.** The app saves a snapshot first, then asks you to sign in.
3. Tap **Sign in with Google**, choose your usual account.
4. Check the Data screen shows your email and a sensible count.
5. Only when the phone looks right, do the same on the tablet. Its data merges in — nothing
   gets replaced.

Your old project keeps its copy. Leave it alone for a week as a safety net.

---

## If something goes wrong

| What you see | Fix |
|---|---|
| "This site isn't allow-listed" | Firebase → Authentication → Settings → Authorized domains → add `yoyo196204-hub.github.io` |
| "Turn on Google sign-in" | Firebase → Authentication → Sign-in method → Google → Enable |
| "The cloud database is blocking access" | Firebase → Firestore → Rules → paste the rules shown in the app → Publish |
| Sign-in seems to do nothing on your phone | Normal for an installed app — it switches to a full-page sign-in. Tap again. |
| "Whose ledger is this?" | Two different Google accounts on one device. Pick **Keep them separate**. |

**Nothing is ever deleted without a backup first.** Data → *Auto-backups on this device* keeps
the last few, and restoring adds things back rather than overwriting.

## Will this cost anything?

No. The free allowance is 50,000 reads and 20,000 writes per day. Triaging a test uses a
handful. Dozens of friends won't get close, and with no card on file it can never charge you.

---

## Starting from scratch (only if you have no project at all)

Your friends don't need this — it's here for completeness.

1. **console.firebase.google.com** → **Create a project** → name it → turn Analytics off → Create.
2. Click the **`</>`** icon → nickname it → Register → copy the `firebaseConfig` block.
3. **Build → Authentication → Get started → Google → Enable** → pick your email → Save.
4. **Build → Firestore Database → Create database → Production mode** → pick a region → Enable.
5. **Rules** tab → replace everything with this → **Publish**:

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

6. **Authentication → Settings → Authorized domains → Add domain** → `yoyo196204-hub.github.io`
7. Put the config from step 2 into `index.html` where it says `const APP_FB_CFG = null;`.
