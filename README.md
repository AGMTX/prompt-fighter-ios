# AI Arena: Prompt Fighter — iOS build via Codemagic (no Mac needed)

This folder wraps the game as a real iPhone/iPad app using **Capacitor**,
a **codemagic.yaml** file that tells Codemagic's cloud Macs how to build,
sign, and upload it, and **RevenueCat** to handle the in-app purchase that
unlocks levels 16–25 (the "Prompt Master Pack"). All from your browser.

## What's in here
- `www/` — the game itself: levels 1–15 are free, levels 16–25 need the
  purchase. `www/index.html` ends with a small RevenueCat bridge script —
  that's the only iOS-specific code in the whole project.
- `package.json`, `capacitor.config.json` — the Capacitor wrapper config
- `codemagic.yaml` — the build recipe Codemagic runs
- `privacy-policy.html` — fill in the two `[FILL IN...]` spots and host it
  somewhere public (GitHub Pages works free — see Step 7)

## Before you start
You'll need the $99/year Apple Developer Program membership (see the
earlier registration steps). Codemagic can't get around that requirement —
Apple requires it to sign and submit any app.

---

## Step 1 — Put this project on GitHub
Codemagic builds from a Git repository, so the project needs a home there.

1. Create a free account at [github.com](https://github.com) if you don't
   have one.
2. Click **+ → New repository**. Name it `prompt-fighter-ios`, keep it
   **Private**, and click **Create repository**.
3. On the new repo's page, click **uploading an existing file**, then drag
   this whole `prompt-fighter-ios` folder's contents in (or use GitHub
   Desktop / `git push` if you're comfortable with git). Commit the files.

## Step 2 — Create the app record in App Store Connect
1. Go to [appstoreconnect.apple.com](https://appstoreconnect.apple.com) →
   **My Apps → +  → New App**.
2. Platform: iOS. Name: "AI Arena: Prompt Fighter" (or your choice).
3. Bundle ID: create a new one matching `com.austinwg04.promptfighter`
   (or change it — see the note in Step 5 below if you pick your own).
4. SKU: anything unique, e.g. `promptfighter001`.
5. Once created, open the app and note the **Apple ID number** shown in
   the top-left "App Information" section (a 10-digit number) — you'll
   need it in Step 5.

## Step 3 — Create the in-app purchase product
This is the "Prompt Master Pack" (levels 16–25) itself, on Apple's side.

1. In App Store Connect, open your app → **Features → In-App Purchases**
   (called "Monetization" on some accounts).
2. Click **+**, choose **Non-Consumable**.
3. Reference Name: `Prompt Master Pack`. Product ID: `prompt_master_pack`
   (you'll use this exact ID again in Step 6).
4. Set a price tier (the game shows $4.99 by default — see the `PRICE`
   variable near the top of the game's script if you want a different
   number; keep the two in sync).
5. Fill in the display name and description, and add a review screenshot
   showing the locked levels 16–25 screen (a simple screenshot of the
   level-select page works).
6. Apple also needs your **banking and tax info** active before a paid
   in-app purchase can go live — do this now under **Agreements, Tax, and
   Banking** in App Store Connect, since it can take a day or so to
   process. You can still build and test with TestFlight before it's
   active; you just can't collect real money until it is.

## Step 4 — Get an App Store Connect API key
This lets Codemagic sign and upload the app without you touching Xcode.

1. In App Store Connect, go to **Users and Access → Integrations → App
   Store Connect API**.
2. Click **+** to generate a new key. Name it "Codemagic", role
   **App Manager**.
3. Download the `.p8` key file **immediately** — Apple only lets you
   download it once. Also note the **Key ID** and **Issuer ID** shown on
   that page.

## Step 5 — Set up RevenueCat (handles the purchase for you)
RevenueCat is a free service that wraps Apple's in-app purchase system so
you don't have to write low-level StoreKit code — it's what `www/index.html`
already talks to.

1. Sign up free at [revenuecat.com](https://www.revenuecat.com).
2. Create a new Project, then add an **App** for iOS, and paste in your
   bundle ID (`com.austinwg04.promptfighter`) and the App Store Connect API
   key from Step 4 (RevenueCat can reuse the same key).
3. Go to **Products**, click **+**, and enter the exact Product ID from
   Step 3 (`prompt_master_pack`). RevenueCat will pull in its price
   automatically once Apple has approved it.
4. Go to **Entitlements**, create one called `premium`, and attach the
   `prompt_master_pack` product to it.
5. Go to **Offerings → default**, add a **Package**, and attach the
   `prompt_master_pack` product to it. This is what the app actually asks
   for when someone taps "Unlock".
6. Go to **API Keys**, copy the **Apple App Store** public API key.
7. Open `www/index.html`, find the line near the bottom that says:
   ```js
   var REVENUECAT_API_KEY = 'YOUR_REVENUECAT_IOS_API_KEY';
   ```
   and paste your key in. Commit and push this change to GitHub.

## Step 6 — Fill in your build details
Open `codemagic.yaml` (either in GitHub's web editor or on your computer,
then re-upload) and edit:

- `bundle_identifier` (two places) — must match what you created in Step 2
- `APP_STORE_APPLE_ID` — the 10-digit number from Step 2

If you changed the bundle ID, also update `appId` in
`capacitor.config.json` to match.

## Step 7 — Sign up for Codemagic and connect everything
1. Go to [codemagic.io](https://codemagic.io) and sign up (you can use
   your GitHub account to sign in, which also connects your repos).
2. Click **Add application**, pick GitHub, and select the
   `prompt-fighter-ios` repo you created in Step 1.
3. When asked what kind of project it is, choose **Capacitor** (or "Other"
   if Capacitor isn't listed — either way it will pick up your
   `codemagic.yaml`).
4. Go to **Teams → your team → Integrations → Apple Developer Portal**.
   Click **Connect**, choose **App Store Connect API key**, and enter:
   - Key name: `codemagic` (must match `app_store_connect: codemagic` in
     `codemagic.yaml`)
   - Issuer ID and Key ID from Step 4
   - Upload the `.p8` file from Step 4

## Step 8 — Host the privacy policy (needed before you can submit)
The easiest free option is GitHub Pages:
1. Fill in the two `[FILL IN...]` spots in `privacy-policy.html`.
2. In your GitHub repo, go to **Settings → Pages**, set the source to your
   main branch, and save.
3. GitHub gives you a public URL like
   `https://yourname.github.io/prompt-fighter-ios/privacy-policy.html` —
   you'll paste this into App Store Connect's "Privacy Policy URL" field
   later, in the app listing.

## Step 9 — Run your first build
1. Back in Codemagic, open your app and click **Start new build**.
2. Pick the `ios-workflow`. It should take roughly 10-20 minutes.
3. Watch the build log. The first run is the most likely to hit a snag
   (usually a signing or bundle-ID mismatch) — Codemagic's log tells you
   exactly which step failed, and their docs/support chat are good at
   walking through fixes.
4. On success, the build is automatically uploaded to **TestFlight**.
   Open the TestFlight app on your own iPhone/iPad, and it will appear
   there within a few minutes for you to install and play.
5. In TestFlight, try both sides: play through level 15 for free, then tap
   **Unlock the Pack** and confirm the purchase flow opens Apple's real
   payment sheet (TestFlight builds use "sandbox" purchases — you won't be
   charged real money, and you can create a free Sandbox Tester account
   under **Users and Access → Sandbox** in App Store Connect to test with).

## Step 10 — Submit for review
Once you've played it on your own device via TestFlight and you're happy:
1. In App Store Connect, fill out the rest of the listing — screenshots,
   description, age rating, category, and the Privacy Policy URL from
   Step 8.
2. Attach the in-app purchase from Step 3 to this version and submit it
   for review **together** with the app binary — Apple reviews a first
   in-app purchase alongside the app, not separately.
3. Attach the TestFlight build to a new version and click **Submit for
   Review**.

You can also flip `submit_to_app_store: true` in `codemagic.yaml` later so
future builds go straight to review automatically — leave it `false` for
now while you're still testing.

## Costs recap
- Apple Developer Program: $99/year (required, paid to Apple)
- Codemagic: free for 500 build minutes/month, which is plenty for an app
  this size (a full build is roughly 10-20 minutes)
- RevenueCat: free up to $2,500/month in tracked revenue, far more than
  this app needs to start
- Apple takes a cut of each sale (15% under the Small Business Program,
  which almost any first-time solo developer qualifies for automatically)

## Testing the paywall on the web version
The Cloudflare/artifact version of the game (not this iOS build) shows the
same locked levels 16-25 and the same "Unlock the Pack" card, but real
purchases only work inside the iOS app — the web version tells the player
that plainly. If you ever want to unlock it there for your own testing,
add `?demo=1` to the URL and a "Demo unlock (testing only)" button appears
on the paywall — it's clearly labeled and never shown to normal visitors.
