---
title: Time machine with Raspberry Pi
date: "2020-04-02T22:12:03.284Z"
template: "post"
draft: false
slug: "/posts/Time-machine-with-Raspberry-Pi"
category: "Homelab"
tags:
  - "NAS"
  - "Raspberry-Pi"
  - "OpenMediaVault"
  - "smb"
description: "How I set up Time Machine on OMV5 with a Raspberry Pi 4"
---

From my [previous post](/posts/My-own-NAS-with-Raspberry-Pi), I have set up a NAS with my Raspberry Pi. Now I want to back up my Mac to it with Time Machine. This is made easy with Time Machine compatible with SMB.

## Steps

### 1. Creating a user for time machine backups

I want to create a user for Time Machine to log on to as, and set quotas so it doesn't use up all the space.

Go to *Access Rights Management > User* and click on "+Add"
![Add user 01](/media/2020-04-02---Time-machine-with-Raspberry-Pi/add-user-01.png)

Set the username and password
![Add user 02](/media/2020-04-02---Time-machine-with-Raspberry-Pi/add-user-02.png)

Make sure this user is in the **users** group.
![Add user 03](/media/2020-04-02---Time-machine-with-Raspberry-Pi/add-user-03.png)

Now you should see the user in the list
![Add user 04](/media/2020-04-02---Time-machine-with-Raspberry-Pi/add-user-04.png)

### 2. Manage user quota

Go to *Storage > File systems* and click on the device the Time Machine share is on. Then click on the Quota button.
![user quota 01](/media/2020-04-02---Time-machine-with-Raspberry-Pi/user-quota-01.png)

Now set the quota for the Time Machine user. I heard from a source that setting it to 2 times your mac storage should be enough.
![user quota 02](/media/2020-04-02---Time-machine-with-Raspberry-Pi/user-quota-02.png)

### 3. Creating a SMB share

Go to the *share* tab in *Services > SMB/CIFS* and click **Add**
![smb share 01](/media/2020-04-02---Time-machine-with-Raspberry-Pi/smb-share-01.png)

Click on + to add a new folder
![smb share 02](/media/2020-04-02---Time-machine-with-Raspberry-Pi/smb-share-02.png)

Enter your configurations
![smb share 03](/media/2020-04-02---Time-machine-with-Raspberry-Pi/smb-share-03.png)

Remember to enable Time Machine Support, then save.
![smb share 04](/media/2020-04-02---Time-machine-with-Raspberry-Pi/smb-share-04.png)

Finally, enable SMB
![smb share 05](/media/2020-04-02---Time-machine-with-Raspberry-Pi/smb-share-05.png)

### 4. Connect via Time Machine

Now the Time Machine share will show up in the Time Machine interface.
![Time Machine 01](/media/2020-04-02---Time-machine-with-Raspberry-Pi/Time-Machine-01.png)
