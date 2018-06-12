
# The Business Scenario
The business scenario is bounded to a very specific domain which helps us to scope the problem on the business requirements that needs to be solved. For this example, we took a Global Marketing Company which wants to create marketing campaigns to track Social Media Trending Topics. The solution should be easy to adapt to cover different campaign types and requirements. 

The company will have different marketing departments in different cities around the world creating campaigns for trending topics in different languages. Because they cannot predict trending topics they should be able to quickly create a campaign and have it running in minutes. 

These marketing campaigns have the objective of track the interaction of people in social media with a certain topic and reward the most engaged participants in the campaign. 

New departments might be added dynamically and they should share some common infrastructural pieces which allows top executive to see the data about their global operations.

Each campaign should be scaled independently based on the level of participation and target audience.

![Scenario](/scenario.png)

The Diagram shows a Social Media Feed (Twitter) as input for a set of campaigns that will all process each Tweet to understand if it matches with the campaign criteria and then it will interact with surrounding services to perform Sentiment Analysis, Rankings and Rewards. 

Each of the individual blocks, such as campaigns and the external services, can be scaled independently.

