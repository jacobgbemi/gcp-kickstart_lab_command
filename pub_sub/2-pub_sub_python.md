### Task 1: Create a virtual environment
1. Create environment and activate
	```
	apt-get install -y virtualenv
	python3 -m venv venv
	source venv/bin/activate
	```

### Task 2: Install the client library
1. install the client library:
	```
	pip install --upgrade google-cloud-pubsub
	```
2. Get the sample code by cloning a GitHub repository:
	```
	git clone https://github.com/googleapis/python-pubsub.git
	```
3. Navigate to the directory:
	```
	cd python-pubsub/samples/snippets
	```

### Task 3: Pub/Sub - Create a topic
1. To publish data to Cloud Pub/Sub you create a topic and then configure a publisher to the topic.
	```
	echo $GOOGLE_CLOUD_PROJECT
	cat publisher.py
	```
2. For information about the publisher script:
	```
	python publisher.py -h
	```
3. Run the publisher script to create Pub/Sub Topic:
	```
	python publisher.py $GOOGLE_CLOUD_PROJECT create MyTopic
	```
4. This command returns a list of all Pub/Sub topics in a given project:
	```
	python publisher.py $GOOGLE_CLOUD_PROJECT list
	```

### Task 4: Create a subscription
1. Create a Pub/Sub subscription for topic with subscriber.py script:
	```
	python subscriber.py $GOOGLE_CLOUD_PROJECT create MyTopic MySub
	```
2. This command returns a list of subscribers in given project:
	```
	python subscriber.py $GOOGLE_CLOUD_PROJECT list-in-project
	```
3. For information about the subscriber script:
	```
	python subscriber.py -h
	```

### Task 5: Publish messages
1. Publish the message "Hello" to MyTopic:
	```
	gcloud pubsub topics publish MyTopic --message "Hello"
	```
2. Publish a few more messages to MyTopic
	```
	gcloud pubsub topics publish MyTopic --message "Publisher's name is <YOUR NAME>"
	gcloud pubsub topics publish MyTopic --message "Publisher likes to eat <FOOD>"
	gcloud pubsub topics publish MyTopic --message "Publisher thinks Pub/Sub is awesome"
	```
3. Use MySub to pull the message from MyTopic:
	```
	python subscriber.py $GOOGLE_CLOUD_PROJECT receive MySub
	```

