Many of our users have problems with iOS signing involving Fastlane. I believe
that we don't see the same issues they do because we let Xcode manage our app
signing, which is a much less common setup among professional app development
teams. I'm going to try setting up signing using fastlane's
[match](https://docs.fastlane.tools/actions/match/) and see what the pain points
are. The resulting app will live in
https://github.com/shorebirdtech/fastlane_demo.

Following
https://medium.com/revelo-tech/setting-up-automatic-ios-release-with-fastlane-and-match-on-ci-cd-server-16c3f1d79bc5

I attempted to create an apple dev account for contact@. However, this was
unsuccessful because their signup flow would not let me enter a phone number for
2FA. Instead, I am just going to use the certs that already exist and create new
ones with bryan@ if needed, as this is just a proof-of-concept.

Creating a private git repo to store the certs
https://github.com/shorebirdtech/AppleCerts. It's currently empty.

Upgraded fastlane. Following setup guide at https://docs.flutter.dev/deployment/cd#fastlane

The app doesn't build for iOS yet due to signing, that's fine. Going to start
the setup on that platform, as it seems to be the most problematic.

## iOS

Ran `fastlane init`. Opted for manual setup, as the other options required
signin to Apple and I don't think I want to do that as per the guide.

- Filled out fields in ios/Appfile
- Ran `fastlane match init`
- Added `dotenv` gem
- Added `.env.default` and `.env.default.sample`
- Filled out env.default file following the "For CD" instructions.
- Ran `fastlane match nuke distribution`
- Ran `fastlane ios generate_new_certificates`
- Added sync_code_signing step for appstore (distribution) cert

Note that https://github.com/shorebirdtech/AppleCerts now contains info for
our certs and provisioning profiles for the `dev.shorebird.fastlane-demo` app.

- Created an apple API key at https://appstoreconnect.apple.com/access/integrations/api
- Set up fastlane deploy script
- Running `fastlane ios deploy`
  - Failed, needed to create an App entry in Appstore Connect
  - Created, now failing at `upload_to_app_store` step. "No data"
  - Ran `flutter build ipa --release`, error is "exportArchive "Runner.app" requires a provisioning profile."
    - Downloaded profile from app store connect, didn't help
    - Ran "Download Manual Profiles" from Xcode, didn't help
    - I am able to upload the Runner.xcarchive file via Xcode, so this appears
      to just be a "fastlane finding the provisioning profile" issue
  - This failure turned out to be because the app's store metadata was not
    properly configured in App Store Connect.
- Disabled automatic incrementing of version numbers in the build_app step

### Adding Shorebird

- Ran `shorebird init`
