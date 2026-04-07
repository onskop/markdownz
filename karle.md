


This is the final boss of mobile development: **iOS Code Signing**. Apple’s security is notoriously strict, and moving away from FlutterFlow (which hid all this pain from you) means your friend now has to face it head-on.

Because you invited him to your team, he has access, but we need to set up his Mac to prove to Apple that *his* specific computer is authorized to compile *your* app.

Here is the exact, step-by-step guide you can send to your friend. 

***

### ⚠️ IMPORTANT PREREQUISITE: The "Developer" Role
Before your friend starts, you (the account owner) need to log into **App Store Connect** and change his role from "Developer" to **"App Manager"**. 
* **Why?** A "Developer" can compile code for a simulator, but they usually *cannot* create the "Distribution Certificates" required to upload an app to the App Store. Changing him to App Manager will make Step 4 happen automatically instead of throwing errors.

***

### Step 1: Prep the MacBook (The Heavy Downloads)
Your friend needs the Apple "engine" installed.
1. **Install Xcode:** Go to the Mac App Store and download **Xcode**. (It is massive, like 12GB+, so start this immediately). Once downloaded, **open it at least once** so it can install its command-line tools.
2. **Install CocoaPods:** This is the tool that links Firebase and other plugins to the iOS app. Open the Mac Terminal and type:
   ```bash
   sudo gem install cocoapods
   ```
   *(It will ask for his Mac login password. It won't show characters as he types, just type it and hit Enter).*

### Step 2: Prepare the Flutter Project
Your friend should open the project folder in Cursor on his Mac.
1. Open the terminal in Cursor.
2. Download all the Flutter packages:
   ```bash
   flutter pub get
   ```
3. Link the iOS specific packages (this generates a crucial file called `Runner.xcworkspace`):
   ```bash
   cd ios
   pod install
   cd ..
   ```

### Step 3: Connect Xcode to Your Apple Team
Now we tell his Xcode who he is.
1. Open **Xcode**.
2. Go to the top menu bar: **Xcode > Settings** (or Preferences) **> Accounts**.
3. Click the **`+`** button in the bottom left, select **Apple ID**, and have him log in with the email you invited to your team.
4. He should now see your team name (e.g., "FitAid LLC") listed on the right side.

### Step 4: The Code Signing Magic
This is where FlutterFlow used to do the heavy lifting. We have to configure the app to use his new credentials.
1. Still in Xcode, go to **File > Open**. 
2. Navigate to your project folder -> `ios` -> and open the file named **`Runner.xcworkspace`** *(Do NOT open `Runner.xcodeproj`, it must be the workspace file)*.
3. On the far left panel, click on the blue folder icon named **Runner**.
4. In the main window, click the **Signing & Capabilities** tab.
5. Check the box that says **"Automatically manage signing"**.
6. In the **Team** dropdown, he must select your team (e.g., "FitAid Team").
7. Ensure the **Bundle Identifier** exactly matches what you have in the App Store (e.g., `com.fitaid.app`).

**🚨 THE LIKELY ERROR HERE:**
Because FlutterFlow created the original "Distribution Certificate", Apple knows it exists, but your friend's Mac doesn't have the physical file. Xcode might show a red error saying: *"Your account already has a valid iOS Distribution certificate, but it is not installed locally."*
* **The Fix:** In Xcode, go back to **Settings > Accounts**. Select your Team, click **Manage Certificates** in the bottom right. Click the **`+`** button and select **Apple Distribution**. This forces Apple to issue a *second* key directly to his Mac. The red error in the main window should disappear.

### Step 5: Compile the Release Build
Once the red errors in Xcode are gone, he doesn't actually need to use Xcode to build. He can go back to **Cursor**.
1. Open the terminal in Cursor.
2. Run the ultimate build command:
   ```bash
   flutter build ipa --release
   ```
*This command will compile the Dart code, assemble the iOS package, sign it with the certificates from Step 4, and create an `.ipa` file (the Apple equivalent of an `.aab` or `.apk`).*

### Step 6: Upload to the App Store (TestFlight)
The terminal will tell you where it saved the file (usually `build/ios/ipa/FitAid.ipa`), but the easiest way to upload it is using an Apple app.

1. Tell your friend to download the **Transporter** app from the Mac App Store (it's an official Apple app).
2. Open Transporter and log in with his Apple ID.
3. Drag and drop the **`FitAid.ipa`** file from the `build/ios/ipa/` folder directly into the Transporter window.
4. Click **Deliver**.

Transporter will check the code, verify the signatures, and push it directly to your App Store Connect account. Within 10-15 minutes, it will show up in **TestFlight** ready for you to test on your iPhone, or to submit to the App Store for review!
