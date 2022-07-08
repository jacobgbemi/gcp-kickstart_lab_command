### Task 1. Upload an object into your bucket
1. To download this image (ada.jpg) into your bucket, enter this command into Cloud Shell:
	```
	curl https://upload.wikimedia.org/wikipedia/commons/thumb/a/a4/Ada_Lovelace_portrait.jpg/800px-Ada_Lovelace_portrait.jpg --output ada.jpg
	```
2. Use the gsutil cp command to upload the image from the location where you saved it to the bucket you created:
	```
	gsutil cp ada.jpg gs://YOUR-BUCKET-NAME
	```
3. Now remove the downloaded image:
	```
	rm ada.jpg
	```

### Task 2. Download an object from your bucket
1. Use the gsutil cp command to download the image you stored in your bucket to Cloud Shell:
	```
	gsutil cp -r gs://YOUR-BUCKET-NAME/ada.jpg .
	```

### Task 3. Copy an object to a folder in the bucket
1. Use the gsutil cp command to create a folder called image-folder and copy the image (ada.jpg) into it:
	```
	gsutil cp gs://YOUR-BUCKET-NAME/ada.jpg gs://YOUR-BUCKET-NAME/image-folder/
	```

### Task 4. List contents of a bucket or folder
1. Use the gsutil ls command to list the contents of the bucket:
	```
	gsutil ls gs://YOUR-BUCKET-NAME
	```

### Task 5. List details for an object
1. Use the gsutil ls command, with the -l flag to get some details about the image file you uploaded to your bucket:
	```
	gsutil ls -l gs://YOUR-BUCKET-NAME/ada.jpg
	```

### Task 6. Make your object publicly accessible
1. Use the gsutil acl ch command to grant all users read permission for the object stored in your bucket:
	```
	gsutil acl ch -u AllUsers:R gs://YOUR-BUCKET-NAME/ada.jpg
	```

### Task 7. Remove public access
1. To remove this permission, use the command:
	```
	gsutil acl ch -d AllUsers gs://YOUR-BUCKET-NAME/ada.jpg
	```

### Task 8: Delete objects
1. Use the gsutil rm command to delete an object - the image file in your bucket:
	```
	gsutil rm gs://YOUR-BUCKET-NAME/ada.jpg
	```

