Task 1: Pub/Sub topics
1.  create a topic called myTopic:
	gcloud pubsub topics create myTopic
2. create two more topics; one called Test1 and the other called Test2:
	gcloud pubsub topics create Test1
	gcloud pubsub topics create Test2
3. To see the three topics you just created, run the following command:
	gcloud pubsub topics list
4. Delete Test1 and Test2 by running the following commands:
	gcloud pubsub topics delete Test1
	gcloud pubsub topics delete Test2
5. Run the gcloud pubsub topics list command one more time to verify the topics were deleted:
	gcloud pubsub topics list

Task 2: Pub/Sub subscriptions
1. Run the following command to create a subscription called mySubscription to topic myTopic:
	gcloud  pubsub subscriptions create --topic myTopic mySubscription
2. Add another two subscriptions to myTopic. Run the following commands to make Test1 and Test2 subscriptions:
	gcloud  pubsub subscriptions create --topic myTopic Test1
	gcloud  pubsub subscriptions create --topic myTopic Test2
3. Run the following command to list the subscriptions to myTopic:
	gcloud pubsub topics list-subscriptions myTopic
4. Now delete the Test1 and Test2 subscriptions. Run the following commands:
	gcloud pubsub subscriptions delete Test1
	gcloud pubsub subscriptions delete Test2
5. See if the Test1 and Test2 subscriptions were deleted.
	gcloud pubsub topics list-subscriptions myTopic

Task 3: gcloud pubsub topics list-subscriptions myTopic
1. Run the following command to publish the message "hello" to the topic you created previously (myTopic):
	gcloud pubsub topics publish myTopic --message "Hello"
2. Publish a few more messages to myTopic.
	gcloud pubsub topics publish myTopic --message "Publisher's name is <YOUR NAME>"
	gcloud pubsub topics publish myTopic --message "Publisher likes to eat <FOOD>"
	gcloud pubsub topics publish myTopic --message "Publisher thinks Pub/Sub is awesome"
3. Use the following command to pull the messages you just published from the Pub/Sub topic:
	gcloud pubsub subscriptions pull mySubscription --auto-ack

Task 4: Pub/Sub pulling all messages from subscriptions
1. populate myTopic with a few more messages.
	gcloud pubsub topics publish myTopic --message "Publisher is starting to get the hang of Pub/Sub"
	gcloud pubsub topics publish myTopic --message "Publisher wonders if all messages will be pulled"
	gcloud pubsub topics publish myTopic --message "Publisher will have to test to find out"
2. Run the pull command with the limit flag:
	gcloud pubsub subscriptions pull mySubscription --auto-ack --limit=3

