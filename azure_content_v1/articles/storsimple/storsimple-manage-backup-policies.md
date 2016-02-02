<properties 
   pageTitle="Manage your StorSimple backup policies | Microsoft Azure"
   description="Explains how you can use the StorSimple Manager service to create and manage manual backups, backup schedules, and backup retention."
   services="storsimple"
   documentationCenter="NA"
   authors="SharS"
   manager="carolz"
   editor=""/>

<tags 
   ms.service="storsimple"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="TBD"
   ms.date="12/14/2015"
   ms.author="v-sharos"/>

# Use the StorSimple Manager service to manage backup policies
> [AZURE.SELECTOR]
- [Update 2](../articles/storsimple/storsimple-manage-backup-policies-u2.md)
- [Update 1 & earlier](../articles/storsimple/storsimple-manage-backup-policies.md)



## Overview
This tutorial explains how to use the StorSimple Manager service **Backup Policies** page to control backup processes and backup retention for your StorSimple volumes. It also describes how to complete a manual backup.

The **Backup Policies** page allows you to manage backup policies and schedule local and cloud snapshots. (Backup policies are used to configure backup schedules and backup retention for a collection of volumes.) Backup policies enable you to take a snapshot of multiple volumes simultaneously. This means that the backups created by a backup policy will be crash-consistent copies. This page lists the backup policies, their types, the associated volumes, the number of backups retained, and the option to enable these policies.

The **Backup Policies** page also allows you to filter the existing backup policies by one or more of the following fields:

* **Policy name** – The name associated with the policy. The different types of policies include:

  * Scheduled policies, which are explicitly created by the user.
* Automatic policies, which are created when the default backup for this volume option was enabled at the time of volume creation. These policies are named as VolumeName_Default where Volume name refers to the name of the StorSimple volume configured by the user in the Azure classic portal. The automatic policies result in daily cloud snapshots beginning at 22:30 device time.
* Imported policies, which were originally created in the StorSimple Snapshot Manager. These have a tag that describes the StorSimple Snapshot Manager host that the policies were imported from.

* **Volumes** – The volumes associated with the policy. All the volumes associated with a backup policy are grouped together when backups are created.

* **Last successful backup** – The date and time of the last successful backup that was taken with this policy.

* **Next backup** – The date and time of the next scheduled backup that will be initiated by this policy.

* **Schedules** – The number of schedules associated with the backup policy.


The frequently used operations that you can perform from this page are:

* Add a backup policy 
* Add or modify a schedule 
* Delete a backup policy 
* Take a manual backup 
* Create a custom backup policy with multiple volumes and schedules 

## Add a backup policy
Add a backup policy to automatically schedule your backups. Perform the following steps in the Azure classic portal to add a backup policy for your StorSimple device. After you add the policy, you can define a schedule (see [Add or modify a schedule](#add-or-modify-a-schedule.md)).


<!--author=alkohli last changed: 9/11/15-->

#### To add a StorSimple backup policy

1. On the device **Quick Start** page, click the **Backup Policies** tab. This will take you to the **Backup Policies** page.

2. At the bottom of the page, click **Add**.

3. In the **Add Backup Policy** dialog box, do the following:

    1. Specify a backup policy name that contains between 3 and 150 characters.
    2. Assign one or more volumes to this backup policy. If you are using other cloud service providers, based on your first selection, the drop-down list will show volumes belonging to only that cloud service provider. This will allow you to group volumes belonging to a single cloud service provider for taking a snapshot.
    3. Click the check icon ![check icon](./media/storsimple-add-backup-policy/HCS_CheckIcon-include.png) to save the policy.

The newly added policy will be displayed in the tabular view on the **Backup Policies** page.
 





![Video available](./media/storsimple-manage-backup-policies/Video_icon.png) **Video available**

To watch a video that demonstrates how to create a local or cloud backup policy, click [here](https://azure.microsoft.com/documentation/videos/create-storsimple-backup-policies/).

## Add or modify a schedule
You can add or modify a schedule that is attached to an existing backup policy on your StorSimple device. Perform the following steps in the Azure classic portal to add or modify a schedule.


<!--author=SharS last changed: 9/15/15-->

#### To add or modify a StorSimple backup schedule

1. On the device **Quick Start** page, click the **Backup Policies** tab. This will take you to the **Backup Policies** page.

2. In the tabular listing of the policies, select and click the policy that you want to edit.

3. Under **general**, you can modify the backup policy name.

4. Under **schedules**, click **manage schedules** to add or remove schedules. In the **Manage Schedule** dialog box, do the following:

    1. Choose an existing schedule or create a new schedule.

    2. Select the type of backup as local or cloud snapshot.

    3. Specify the backup frequency, retention, and starting time of the schedule.

    4. Enable or disable the schedule.

    5. Click the check icon ![check icon](./media/storsimple-add-modify-backup-schedule/HCS_CheckIcon-include.png) to save the changes to the schedule.

5. In the **Volumes** section, add or remove the volumes to this policy.

6. Save the changes to this policy.

7. You will prompted for confirmation. Click **OK** to save the policy.

The **Backup Policies** page will be updated to save the changes to the policy.
 



## Delete a backup policy
Perform the following steps in the Azure classic portal to delete a backup policy on your StorSimple device.


<!--author=SharS last changed: 11/06/15-->

#### To delete a StorSimple backup policy

1. On the device **Quick Start** page, click the **Backup Policies** tab. This will take you to the **Backup Policies** page.

2. Select the policy by clicking anywhere in the corresponding row except for the first column, and then click **Delete** at the bottom of the page.

3. You will be prompted for confirmation. Keep in mind that deleting a backup policy will delete all the associated backups. Click **Yes** to delete.

The **Backup Policies** page will be updated to display the new list of policies.
 





## Take a manual backup
Perform the following steps in the Azure classic portal to create an on-demand (manual) backup for a single volume.


<!--author=SharS last changed: 9/15/15-->


#### To create a manual backup

1. On the **Devices** page, go to the **Backup Policies** tab. This tab lists all the backup policies in a tabular format, including the policy for the volume that you want to back up.

2. Select the policy by clicking anywhere in the corresponding row except for the first column. At the bottom of the page, click **Take backup**. The button will expand to show the backup options: local snapshot and cloud snapshot. 

3. When you choose either of these options, you will be prompted for confirmation. Click **Yes**. 

    ![Create manual backup](./media/storsimple-create-manual-backup/HCS_CreateManualBackup1-include.png)
 
    This will start a job to create a snapshot. You will see a notification at the bottom of the page after the job is successfully created.

4. To monitor the job, click **View Job** in the notification area (at the bottom of the page). 

    ![Monitor the manual backup](./media/storsimple-create-manual-backup/HCS_CreateManualBackup2-include.png)

5. After the backup job is finished, go to the **Backup catalog** tab.

6. Set the filter selections to the appropriate device, backup policy, and time range. Click the check icon ![check icon](./media/storsimple-create-manual-backup/HCS_CheckIcon-include.png) after setting the filters.

  The backup should appear in the list of backup sets that is displayed in the catalog.


## Create a custom backup policy with multiple volumes and schedules
Perform the following steps in the Azure classic portal to create a custom backup policy that has multiple volumes and schedules.

<!--author=SharS last changed: 9/15/15-->

#### To create a custom backup policy

1. On the **Devices** page, click **Backup Policies** and then click **Add**.

2. In the **Add a backup policy** dialog box, under **Define your backup policy**:

    1. Specify a backup policy name.

    2. Select the volumes to be added to this policy. You can choose to add multiple volumes by selecting them from the drop-down list.

    3. Click the check icon ![check icon](./media/storsimple-add-backup-policy/HCS_CheckIcon-include.png).

     You will be notified after the policy is created successfully. The backup policies page will also be updated to show the newly created policy.

4. Click the policy name (first column) to drill down into details of the policy you just created.

5. Click **manage schedules**.

6. In the **Manage Schedules** dialog box:

    1. Select **Create new** to add another schedule.

    2. From the drop-down list, choose the backup type as **local** or **cloud** snapshot.

    3. Specify the backup frequency in minutes, hours, days, or weeks.

    4. Select a retention. The retention choices depend on the backup frequency. For instance, for a daily policy, the retention can be specified in weeks, whereas retention for a monthly policy is in months.
 
    5. Select the starting time and date for the policy.

    6. Select the check box to enable the policy.

7. Click the check icon ![check icon](./media/storsimple-add-backup-policy/HCS_CheckIcon-include.png) to finish.

8. You will return to the policy details. Click **Save** to save the changes you made to this policy. You will be notified when the policy has been saved.

9. Navigate back to the **Backup Policies** page. The tabular listing of the backup policies will be updated to display the modified policy.

    ![Custom backup policy](./media/storsimple-create-custom-backup-policy/HCS_CustomBackupPolicyM-include.png).




## Next steps
Learn more about [using the StorSimple Manager service to administer your StorSimple device](storsimple-manager-service-administration.md).

