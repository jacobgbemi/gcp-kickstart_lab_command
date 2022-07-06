Task 1. Initialize App Engine
1. Initialize your App Engine app with your project and choose its region:
	gcloud app create --project=$DEVSHELL_PROJECT_ID
2. Clone the source code repository for a sample application in the hello_world directory:
	git clone https://github.com/GoogleCloudPlatform/python-docs-samples
3. Navigate to the source directory:
	cd python-docs-samples/appengine/standard_python3/hello_world

Task 2. Run Hello World application locally
1. Create a Dockerfile
	touch Dockerfile
2. Edit the Dockerfile [hint: nano Dockerfile] to contain the following content.
	FROM python:3.7
WORKDIR /app
COPY . .
RUN pip install gunicorn
RUN pip install -r requirements.txt
ENV PORT=8080
CMD exec gunicorn --bind :$PORT --workers 1 --threads 8 main:app
3. Build a container image to run a Python virtual environment.
	docker build -t test-python . 
4. Run the application:
	docker run --rm -p 8080:8080 test-python 
5. In Cloud Shell, click Web preview (Web Preview icon) > Preview on port 8080 to preview the application.
6. To end the test, return to Cloud Shell and press Ctrl+C to abort the deployed service

Task 3. Deploy and run Hello World on App Engine
1. Navigate to the source directory:
	cd ~/python-docs-samples/appengine/standard_python3/hello_world
2. Deploy your Hello World application.
	gcloud app deploy
3. Launch your browser to view the app at http://YOUR_PROJECT_ID.appspot.com
4. Run the command and Copy and paste the URL into a new browser window.
	gcloud app browse

Task 4. Disable the application
1. In the Cloud Console, on the Navigation menu (Navigation menu icon), click App Engine > Settings.
2. Click Disable application.
3. Read the dialog message. Enter the App ID and click DISABLE.
