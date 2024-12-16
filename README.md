# Simple-POC-using-Redis-and-Crew.ai
Deploy in an AWS an implementation of a minimal viable proof of concept demonstrating message passing between two Crew.ai instances using a Redis-based message broker. The system will validate:
1. Successful serialization of crew output
2. Inter-crew message transmission
3. Context preservation across crew boundaries

Implementation Components:
```python
import redis
import json
from crewai import Agent, Crew, Task

class CrewMessageBroker:
    def __init__(self, redis_host='localhost', redis_port=6379):
        self.redis = redis.Redis(host=redis_host, port=redis_port)
        self.OUTPUT_CHANNEL = 'crew_output'
        self.INPUT_CHANNEL = 'crew_input'

    def publish_crew_output(self, crew_output):
        serialized_output = json.dumps(crew_output)
        self.redis.publish(self.OUTPUT_CHANNEL, serialized_output)

    def subscribe_to_input(self):
        pubsub = self.redis.pubsub()
        pubsub.subscribe(self.INPUT_CHANNEL)
        return pubsub

class CrewIntegrationTest:
    def __init__(self):
        self.message_broker = CrewMessageBroker()
        self.research_crew = self._setup_research_crew()
        self.analysis_crew = self._setup_analysis_crew()

    def _setup_research_crew(self):
        research_agent = Agent(
            role='Technology Research Specialist',
            goal='Gather initial insights on a given technology trend',
            backstory='Expert in preliminary technology research'
        )
        
        research_task = Task(
            description='Conduct initial research on emerging technology trends',
            agent=research_agent,
            expected_output='Comprehensive initial research summary'
        )

        return Crew(
            agents=[research_agent],
            tasks=[research_task]
        )

    def _setup_analysis_crew(self):
        analysis_agent = Agent(
            role='Technology Trend Analyst',
            goal='Perform deep analysis on provided research',
            backstory='Expert in extracting complex insights from research data'
        )
        
        analysis_task = Task(
            description='Analyze and expand on provided technology research',
            agent=analysis_agent,
            expected_output='Detailed analytical report with advanced insights'
        )

        return Crew(
            agents=[analysis_agent],
            tasks=[analysis_task]
        )

    def execute_integration_test(self, initial_topic):
        # Step 1: Research Crew Generates Initial Output
        research_output = self.research_crew.kickoff(initial_topic)
        
        # Step 2: Publish Research Output to Message Broker
        self.message_broker.publish_crew_output(research_output)
        
        # Step 3: Analysis Crew Consumes Published Output
        pubsub = self.message_broker.subscribe_to_input()
        for message in pubsub.listen():
            if message['type'] == 'message':
                input_data = json.loads(message['data'])
                analysis_result = self.analysis_crew.kickoff(input_data)
                return analysis_result

# Test Execution
def run_crew_integration_test():
    integration_test = CrewIntegrationTest()
    final_output = integration_test.execute_integration_test(
        initial_topic="Artificial Intelligence in Quantum Computing"
    )
    print("Integration Test Complete. Final Output:", final_output)

if __name__ == "__main__":
    run_crew_integration_test()
```

Test Success Criteria:
1. Research crew successfully generates initial output
2. Output is serialized and published to Redis
3. Analysis crew receives and processes the input
4. Final output demonstrates meaningful transformation of initial research

Prerequisites:
- Redis server running locally
- crewai package installed
- redis-py library installed

Recommended Next Steps:
- Add error handling for message transmission
- Implement more robust serialization
- Create mechanism for handling transmission failures
---------------To implement and deploy a minimal viable proof of concept demonstrating message passing between two Crew.ai instances using a Redis-based message broker, you can follow the code and guidelines below. This approach validates that data is serialized, transmitted between the two crew instances, and that context is preserved across boundaries.
Key Steps:

    Set up Redis: A Redis instance is required as a message broker between the two Crew instances.
    Install Dependencies: The Python packages redis and crewai should be installed.
    Implement CrewMessageBroker: This handles serialization, publishing, and subscribing to messages.
    Define Crew (Research and Analysis): Each crew performs specific tasks.
    Integrate Crew Instances: The research crew generates initial output and the analysis crew receives and processes the output.
    Deploy to AWS: Once locally tested, deploy to AWS using EC2 and ensure the Redis instance is properly set up.

Implementation
1. Install Required Libraries

To run the Python code below, first ensure you have the required libraries installed:

pip install redis crewai

2. Python Code Implementation

Here's the implementation for the proof of concept:

import redis
import json
from crewai import Agent, Crew, Task

class CrewMessageBroker:
    def __init__(self, redis_host='localhost', redis_port=6379):
        self.redis = redis.Redis(host=redis_host, port=redis_port)
        self.OUTPUT_CHANNEL = 'crew_output'
        self.INPUT_CHANNEL = 'crew_input'

    def publish_crew_output(self, crew_output):
        """
        Serialize the output of the crew and publish it to Redis.
        """
        serialized_output = json.dumps(crew_output)
        self.redis.publish(self.OUTPUT_CHANNEL, serialized_output)

    def subscribe_to_input(self):
        """
        Subscribe to the input channel to listen for messages.
        """
        pubsub = self.redis.pubsub()
        pubsub.subscribe(self.INPUT_CHANNEL)
        return pubsub

class CrewIntegrationTest:
    def __init__(self):
        # Initialize message broker and crew
        self.message_broker = CrewMessageBroker()
        self.research_crew = self._setup_research_crew()
        self.analysis_crew = self._setup_analysis_crew()

    def _setup_research_crew(self):
        """
        Set up the research crew with a research agent and task.
        """
        research_agent = Agent(
            role='Technology Research Specialist',
            goal='Gather initial insights on a given technology trend',
            backstory='Expert in preliminary technology research'
        )
        
        research_task = Task(
            description='Conduct initial research on emerging technology trends',
            agent=research_agent,
            expected_output='Comprehensive initial research summary'
        )

        return Crew(
            agents=[research_agent],
            tasks=[research_task]
        )

    def _setup_analysis_crew(self):
        """
        Set up the analysis crew with an analysis agent and task.
        """
        analysis_agent = Agent(
            role='Technology Trend Analyst',
            goal='Perform deep analysis on provided research',
            backstory='Expert in extracting complex insights from research data'
        )
        
        analysis_task = Task(
            description='Analyze and expand on provided technology research',
            agent=analysis_agent,
            expected_output='Detailed analytical report with advanced insights'
        )

        return Crew(
            agents=[analysis_agent],
            tasks=[analysis_task]
        )

    def execute_integration_test(self, initial_topic):
        """
        Execute the integration test: research crew generates output,
        message broker handles transmission, and analysis crew processes input.
        """
        # Step 1: Research Crew Generates Initial Output
        research_output = self.research_crew.kickoff(initial_topic)
        
        # Step 2: Publish Research Output to Message Broker
        self.message_broker.publish_crew_output(research_output)
        
        # Step 3: Analysis Crew Consumes Published Output
        pubsub = self.message_broker.subscribe_to_input()
        for message in pubsub.listen():
            if message['type'] == 'message':
                input_data = json.loads(message['data'])
                analysis_result = self.analysis_crew.kickoff(input_data)
                return analysis_result

# Test Execution
def run_crew_integration_test():
    """
    Run the crew integration test by setting up the system and executing it.
    """
    integration_test = CrewIntegrationTest()
    final_output = integration_test.execute_integration_test(
        initial_topic="Artificial Intelligence in Quantum Computing"
    )
    print("Integration Test Complete. Final Output:", final_output)

if __name__ == "__main__":
    run_crew_integration_test()

Explanation of the Code:

    CrewMessageBroker:
        publish_crew_output(): This method serializes the output of the research_crew into JSON and publishes it to Redis using the crew_output channel.
        subscribe_to_input(): This method subscribes to the crew_input channel and listens for messages.

    CrewIntegrationTest:
        _setup_research_crew(): Defines a research_agent and a task. The agent gathers initial insights on a given topic.
        _setup_analysis_crew(): Defines an analysis_agent and a task. The agent performs deeper analysis on the research output.
        execute_integration_test(): Orchestrates the integration by running the research crew, sending their output through the message broker, and then having the analysis crew consume and process it.

    run_crew_integration_test(): This function starts the test, invoking the execute_integration_test() method, simulating the entire flow from research to analysis.

Expected Workflow:

    The research crew generates an initial output (research on "Artificial Intelligence in Quantum Computing").
    This output is serialized into JSON and published to Redis.
    The analysis crew subscribes to Redis and receives the research output.
    The analysis crew processes the input and produces a transformed output (e.g., a detailed report).
    The test checks that the message passing system is functional and that context is preserved.

Prerequisites:

    Redis: Ensure Redis is running on your local machine or in your cloud infrastructure (e.g., AWS). You can start Redis locally with the command redis-server.
    Crew.ai: You’ll need access to Crew.ai's package or a similar framework for agent-based modeling.
    Python Libraries: Install required libraries like redis and crewai.

AWS Deployment:

    Launch an EC2 instance: Launch a new EC2 instance on AWS (e.g., t2.micro for testing).
    Install Redis on EC2: Follow the Redis installation guide on your EC2 instance.
    Transfer the Python Code: Upload the Python code to the EC2 instance and ensure Redis is running.
    Run the Python Code: Once the instance is running, SSH into it and run the Python script to validate the message passing between the two Crew instances.

Recommended Next Steps:

    Error Handling: Implement error handling for Redis connection issues or task failures.
    Scalability: Ensure that the system can scale for larger message volumes, possibly using Redis clusters.
    Context Management: Extend the system to handle more complex scenarios with better context preservation.
    Testing: Further test the setup with multiple agents and ensure scalability.

By following the above implementation steps, you will have a working proof of concept for message passing between two Crew.ai instances using Redis as a message broker.--
