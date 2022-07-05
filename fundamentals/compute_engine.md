### Task 1: Create a virtual machine instance
1. Create virtual machine instance as instance1, with Window Server OS.
2. In the cloud shell, You can list the active account name with this command:
```
gcloud auth list
```
3. In the cloud shell, You can list the project ID with this command:
	gcloud config list project

Task 2: Remote Desktop (RDP) into the Windows Server
1. Test the status of Windows Startup
	gcloud compute instances get-serial-port-output instance-1
2. If prompted, type n and press Enter.

Task 3: RDP into the Windows Server
1. To set a password for logging into the RDP
	 gcloud compute reset-windows-password [instance] --zone us-central1-a --user [username]
2. (Y/n)?, enter Y.
	
