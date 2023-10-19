###########################################################
####    Create a persistent state in Cloud Shell     ######
###########################################################
1. Type in the shell

$ INFRACLASS_REGION=[YOUR_REGION]
$ INFRACLASS_PROJECT_ID=[YOUR_PROJECT_ID]

$ mkdir infraclass
$ touch infraclass/config
$ echo INFRACLASS_REGION=$INFRACLASS_REGION >> ~/infraclass/config
$ echo INFRACLASS_PROJECT_ID=$INFRACLASS_PROJECT_ID >> ~/infraclass/config
$ source infraclass/config

2. Modify the bash profile and create persistence

$ nano .profile

3. Add the following line to the end of the file:

$ source infraclass/config


Note: If your Cloud Shell environment is ever corrupted, instructions on resetting it are in the Cloud Shell Documentation (https://cloud.google.com/shell/docs/resetting-cloud-shell) article titled Disabling or Resetting Cloud Shell. This will cause everything in your Cloud Shell environment to be set back to its original default state.
