
1. In the **Hybrid connections** blade, click the hybrid connection you just created, then click **Listener Setup**.

    ![Click Listener Setup](./media/app-service-hybrid-connections-manager-install/D04ClickListenerSetup.png)

2. The **Hybrid connection properties** blade opens. Under **On-premises Hybrid Connection Manager**, choose **download and configure manually**, save the downloaded HybridConnectionManager.msi package, and copy the gateway connection string.

    ![Click here to install](./media/app-service-hybrid-connections-manager-install/D05ClickToInstallHCM.png)

3. From an administrator command prompt, type the following command to start the installer:

        start HybridConnectionManager.msi
4. After the installer runs, click **Not now**, then browse to the %ProgramFiles%\Microsoft\HybridConnectionManager folder, run HCMConfigWizard.exe and click **Yes** in the **User Account Control** dialog.

5. Paste the hybrid connection string that you copied earlier and click **OK**. 

    ![Installing](./media/app-service-hybrid-connections-manager-install/D08aHCMInstallManual.png)

6. When the install completes, click **Close**.

    ![Click Close](./media/app-service-hybrid-connections-manager-install/D09HCMInstallComplete.png)

    On the **Hybrid connections** blade, the **Status** column now shows **Connected**. 

    ![Connected Status](./media/app-service-hybrid-connections-manager-install/D10HCStatusConnected.png)


