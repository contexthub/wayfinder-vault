WayFinder with Vault
---

An upgraded WayFinder app that lets you store beacon data in the ContextHub Vault which gives you the flexibility to change it any time


## Purpose

WayFinder with Vault takes our previous WayFinder app and adds functionality by storing all beacon data in the vault on the server instead of in a json file so it can be changed on the fly without recompiling the app. This lets you have the latest information sent to devices each time the app is run.

## Setup

If you haven't already, go through the first version of WayFinder for a broader introduction to ContextHub and iBeacons where terminology like UUID, major, and minor values as well as how to set up your own beacons for testing. We will again be using the same 3 beacons from that sample app, so use either real beacons or Locate for iBeacon to set them up:

*  UUID: B9407F30-F5F8-466E-AFF9-25556B57FE6D
- Major: 100
- Minor: 1
- Name: LOBBY
*  UUID: B9407F30-F5F8-466E-AFF9-25556B57FE6D
- Major: 100
- Minor: 2
- Name: IDEAWALL
*  UUID: B9407F30-F5F8-466E-AFF9-25556B57FE6D
- Major: 100
- Minor: 3
- Name: TEAMROOM

You will need to create your own ContextHub account for this demo, so log into the portal and have an app ID ready to use. In order for any events to be triggered (geofence_in/out, beacon_in/out/changed, etc..), there must be a contextual rule on the server. Contextual rules facilitate additional actions that can be performed when ContextHub received a contextual event from a device. For right now, we don't need any additional processing, so these context rules will be simple. If you need more help setting up contexts, refer to the "Hello ContextHub" example for more info.

1. In the "Contexts" tab, click, "New Context" and create 3 new contexts for "beacon_in", "beacon_out", and "beacon_changed". In the code box, write "true" to indicate the result of this rule is always true.
2. Click on the "Beacons" tab to access all of the beacons for your app. Beacons can be created both on the server and through the SDK using the `CCHBeaconService` service. Recreate the same beacons from above in the setup section on the ContextHub server, making sure to include the major and minor values. Give them all the tag "beacon-wayfinder".
3. Click on the vault tab in ContextHub to see the data you have stored in the vault. The vault is where you can store data to access in your app at any time in json format. Go ahead and and click the "Add data" button, then add 3 vault items with the tag "vault-wayfinder" that are inside `wayfinderBeaconVaultData.json` in the Xcode project.
4. At this point, all server side data on ContextHub is setup. Remember to add your individual app id to your app delegate then your app should be ready to go.


### Demo

Launch and run the app from your device. Here's how it works:

Device (unchanged from vanilla WayFinder):

1. The app will wait until it detects any beacons are in range to display the welcome screen. 
2. The app will wait until it detects the first "LOBBY" beacon specifically in close range (~1-2 ft) before showing the "Start Tour" button.
3. The user should tap the "Start Tour" button to advance to a map of the next location.
4. The app should respond to the close proximity of the 2nd "IDEAWALL" beacon and advance automatically as if you were walking around our actual office.
5. The user again should tap the "Next Stop" button to advance to a map of the next location.
6. The app should respond to the close proximity of the 3rd "TEAMROOM" beacon and again advance automatically with info about the TEAM ROOM.
7. The user then finishes the tour and the app shows a screen with the number of times the tour has been completed.

So that's pretty much the same. But this app is about changing the data in the server without recompiling so lets do that now.
ContextHub:

8. Go back to the vault section in the developer portal, click on the gear icon, and edit the "Lobby" item. Change the text after "locationDescription" to something new like "YOU'RE AT THE ENTRANCE", then click update vault.

Device:
9. Back on your device, close and rerun the app and notice the information change!

Now you are free to change and update information in the demo without changing the app in the device. This demo is also smart enough that you can add more beacons and associated vault items to make your WayFinding trip even longer (you'll need additional map images of course)!


### Code

Building on the first WayFinder, the critical part of this demo is grabbing beacon data from off the server to populate your local beacon store:

1. To grab beacons from off the server and check if you have any beacons, call `[[CCHBeaconService sharedService] getBeaconsWithTags:@[@"wayfinder"] completionHandler:^(NSArray *beacons, NSError *error)`. `NSArray *beacons` will have a count of the number of beacons with that tag (it could be zero).
2. Loop through each of the beacons and call `[CCHBeaconService regionForBeaconInfo:]` on a beacon item in the array in order to get a CLBeaconRegion that contains the proximityUUID, identifier, major, and minor values.
3. To grab vault data associated with each beacon, you use the CCHVault. Call `[[CCHVault sharedService] getItemsWithTags:@"vault-wayfinder" completion:^(NSArray *carbonResponses, NSError *error){}];` which will grab the vault items in a container. Then we associate the beacon data with the data from the vault through an `NSPredicate` search on the name parameter. Look at the code in WFBeaconStore and WFBeaconMetadata for more info.


That's it! Hopefully this demo showed you how storing data in the vault adds a more dynamic feel to your application by keeping information up to date without needing to change anything inside the app.


Note: Current demo is set up such that a response from a beacon is triggered when it is either "near" the device (6-12 inches) or "immediate" to the device (within 6 inches). It takes approximately 2-3 seconds to notice a transition in state from far (within 50 ft) to other states (near or immediate).
