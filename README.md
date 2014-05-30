autoenroll
==========

This is a proof of concept workflow to auto enroll clients, or self heal a client if the binary or any other mechanism that is required for a device to communicate and authenticate to the JSS.   

The proposed workflows are as follows:

1 - have a policy that runs on the enrollment trigger, that payloads the script and launchd, and at the end of the policy in the run command box, use launchctl to load the loaunchd item.

2 - have this also a part of your post image workflows.

Please note this is a work in progress, and a proof of concept.  Many differnet orgs will have different needs, cultures, and policies in place that may change how you would run something like this.  That being said, this is mainly designed to self heal a client when something goes wrong.   If your users have local admin rights and are smart, they will always win.   Keep that in mind.

Also the auto_reenroll.sh is currently not tested.   It is still POC.

How this works:

There is a launchd that runs at system start up, and every hour, and it simply runs a script that tries to execute a Casper policy.  This Casper policy simply touches a file on the local file system.   This policy allows us to confrim a few things.  First and foremost it allows us to confirm that certificate based authentication is working properly with the device.   It also proves to us there is a valid device record in the Casper database.  If the JAMF.keychain were to become corrupted, or not working, this would fail.  If someone were to delete the device record in the JSS, this would not work.   If the binary/framework were compromised or broken, this would not work.   So, this simple policy execution verifies this all works.   

For my POC, I created some folders in a package in Composer.  These folders were as follows:

/private/var/client
/private/var/client/downloads
/private/var/client/receipts
/private/var/client/pkg

In the /private/var/client folder sat my script that would test communication and re-enroll if needed.   The downloads folder was used to curl down a quickadd.pkg if it was not present.   The receipts folder is where I touch the files for a dummy receipt system via Casper policy and the scripts.   The pkg folder simply has a functional quickadd.pkg that is cached and deployed via the enrollment policy.   So, whenever a device gets enrolled, this package payloads these folders, files, and the quickadd.pkg.   These folders could technically be anywhere you desired, this is just what I came up with for my POC.   

The launch daemon also has a watchpath in it from the JAMF binary.  This means anytime the JAMF binary is modified it will trigger the script.  This means if someone decides to remove the framework, or even do say a chmod -x to the binary this will trigger the script.  It would also be triggered during an upgrade.  So, in the rare event an upgrade botches, the script would be triggered instantly.   As long as the test policy can be ran, it just exits because everything is working.


Current Caveats:

- if a device is re-enrolled the device record would be overwritten, flushing all the one time logs.  Some polciies may be reran at that time.
- Overwriting the current device record would also get rid of the device's historical data, this could be important to you or your org.
- The current tested POC uses quickadds, the non tested POC just curls down the binary
- Need to look at leveraging curl over ping to check if the JSS is up and can be authenticated to, ping could cause false positives
- launchd item must not have the name JAMF in it.  A remove frameowrk would wild card JAMF launch daemons and remove it.


Requirements:

- manaual trigger policy that touches a file in /private/var/client/receipts from the run command box 
- package of the launchd and script
- policy to deploy the launchd package with a launchctl command in the run command field in the policy to load the launchd after it payloads


This is a proof of concept, and not a fully supported soltuion from JAMF.   This also needs to be tested in your test envrinonment before going into production.  Any forks or commits from other customers would be highly apprecaited and would help us better understand how to improve the soltuion.  You will know your own infrastructure and org culture than we will, but we would love to improve this soltuion to help everyone.   


Originally developed by t-lark
