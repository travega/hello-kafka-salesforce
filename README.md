Salesforce PushTopics to Kafka
------------------------------

This simple app uses the Salesforce Streaming API to listen for events in Salesforce and then sends them to Kafka.

## Cloud Setup

1. [Signup for a Salesforce Developer Org](https://developer.salesforce.com/signup)
2. In Salesforce, create a PushTopic using the Execute Anonymous Apex feature in the Developer Console:

        PushTopic pushTopic = new PushTopic();
        pushTopic.Name = 'chatter';
        pushTopic.Query = 'SELECT Id, Name FROM Contact';
        pushTopic.ApiVersion = 36.0;
        pushTopic.NotifyForOperationCreate = true;
        pushTopic.NotifyForOperationUpdate = true;
        pushTopic.NotifyForOperationUndelete = true;
        pushTopic.NotifyForOperationDelete = true;
        pushTopic.NotifyForFields = 'Referenced';
        insert pushTopic;

3. [![Deploy on Heroku](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)
4. Add TOPIC environment variable

        heroku config:set TOPIC=chatter -a <YOUR_APP>

5. Add the Heroku Kafka Addon to the app

        heroku addons:add heroku-kafka -a <YOUR_APP>
   
   If you are using this app as part of the Dotnet kafka consumer app then attach the Kafka instance created there

        heroku addons:attach <OTHER_APP_NAME>::KAFKA -a <THIS_APP_NAME> 

6. Install the Kafka plugin into the Heroku CLI

        heroku plugins:install heroku-kafka

7. Wait for Kafka to be provisioned:

        heroku kafka:wait -a <YOUR_APP>

8. If not create as part of the Dotnet consumer app, add a new Kafka topic:

        heroku kafka:topics:create chatter --partitions 32 -a <YOUR_APP>

9. Watch the Kafka log

        heroku kafka:topics:tail chatter -a <YOUR_APP>

10. Make a change to a Contact in Salesforce and you should see the event in the Kafka log.


## Local Setup

> This uses the same Kafka system as above.

1. Clone the source:

        git clone https://github.com/travega/hello-kafka-salesforce

1. Setup a `.env` file with the necessary info:

        heroku config -s > .env
        echo "SALESFORCE_USERNAME=<YOUR SALESFORCE USERNAME>" >> .env
        echo "SALESFORCE_PASSWORD=<YOUR SALESFORCE PASSWORD>" >> .env
        set -o allexport
        source .env
        set +o allexport

1. Run the app:

        ./activator ~run
