## Proactive notification standalone sender

This project aims at simplifying sending proactive events to a skill from an external standalone process. 

For each one of the skills configured in [```skills.json```](./skills.json), this code performs the following operations:
1. Obtains an authentication token from Alexa by using the skill's ```client_id``` and ```client_secret```. You can obtain these in the Build Tab - Permissions section of your skill in the Alexa developer console (if they are not visible try enabling a permission such as reminder and then disable it).
2. Uses the authentication token obtained in the previous step to *multicast* notifications to those users of the skill that granted permissions to notifications in the Alexa app. Sends events following the [```AMAZON.MessageAlert.Activated```](https://developer.amazon.com/docs/smapi/schemas-for-proactive-events.html#message-alert) schema. You can edit the event template in [```message-template.json```](./message-template.json) if you need to. The property ```messageAlert.event.payload.messageGroup.creator.name``` in the template is overridden at runtime using the ```message``` property provided in [```skills.json```](./skills.json) so be careful if you change to a different schema as this will result in an error.

[```skills.json```](./skills.json) is an array of objects with the following properties:

* ```skill_name```: this will only be used for logging purposes.
* ```client_id```: the client_id obtained from the "Permissions" tab of Alexa developer console.
* ```client_secret```: the client_secret obtained from the "Permissions" tab of Alexa developer console.
* ```message```: the message you want to send to users which will be sent as the name of the sender.
* ```validity_hours```: number of hours during which this message will be considered valid (defaults to 24).

### Adding proactive events to your skill

Remember that if you want to send proactive events to your skill, you need to modify the skilll manifest (skill.json) to add notification permissions to the skill and declare the schema(s) of the proactive events your skill is allowed to send. Using SMAPI, you need to:

1. Add the ```permissions``` object to the skill's ```manifest``` property (skill.json):

<p>

    "permissions": [
        {
            "name": "alexa::devices:all:notifications:write"
        }
    ]

2. Add the ```events``` object to the skill's ```manifest``` property:

<p>

    "events": {
      "publications": [
        {
          "eventName": "AMAZON.MessageAlert.Activated"
        }
      ],
      "endpoint": {
        "uri": "** TODO: REPLACE WITH YOUR Lambda ARN after created **"
      },
      "subscriptions": [
        {
          "eventName": "SKILL_PROACTIVE_SUBSCRIPTION_CHANGED"
        }
      ]
    }

3. Redeploy your skill by running:

<p>

    ask deploy -t skill

If you want to declare that the skill accepts more than one schema, just add them into ```events.publications``` above and remember to change the template in ```message-template.json```.

The file [```skill.json```](./skill.json) contains sample skill manifest for your reference.

The process is more thoroughly described in the official [Alexa Proactive Events API documentation](https://developer.amazon.com/docs/smapi/proactive-events-api.html#onboard-smapi). 

You can find a working example on how to configure your skill for Proactive Events in the official Alexa Github repository: https://github.com/alexa/alexa-cookbook/tree/master/feature-demos/skill-demo-proactive-events

### Prerequisites

You need Node.js >= 10.x to run the code and NPM to install the required dependencies.

How to install these two tools goes beyond the scope of this document.

### Setup

To download project dependencies simply run the following in the project root:

    npm install

### How to run

You need to pass the following command line arguments:
* ```environment```: whether the target events will be sent to the ```live``` or ```development``` endpoints. Allowed values are ```dev``` and ```pro```.
* ```region```: identifies the region of the Alexa endpoint to use to send proactive events. Allowed values are ```EU``` (Europe), ```NA``` (North America) and ```FE``` (Far East). **Remember**: if your users are located in NA and you are sending events trough the EU endpoint, users located in NA won't receive any notification. If you're using an Alexa Hosted Skill set the endpoint to NA.
* ```message```: this is optional and lets you override the message property defined in [```skills.json```](./skills.json). If you pass the message through this argument, the same message will be sent to all skills configured in [```skills.json```](./skills.json).

Once you've configured your skill ```client_id``` and ```client_secret``` in [```skills.json```](./skills.json), you can just run the project as:

    node index.js -e <environment> -r <region> -m <message>

For example:

    node index.js -e dev -r EU


