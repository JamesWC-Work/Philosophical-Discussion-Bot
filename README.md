# Philosophical Discussion Bot

Final project for the University of Helsinki course: Building AI.

## Summary

The project aims to create a conversational AI bot that can discuss philosophical concepts and theories with users. The bot can engage in discussions, provide insights, and ask questions to promote critical thinking and learning.

## Background

Philosophy is an interesting and important field that deals with fundamental questions about existence, knowledge, reality, and morality. However, it can also be intimidating for beginners and require a lot of time and effort to understand. The project aims to solve this problem by creating a conversational AI bot that can engage with users in discussions about philosophy. The bot can help users to learn and think critically about philosophical concepts and theories in a user-friendly and interactive way.

## How is it used?

The bot can be accessed via a web or mobile application, where users can initiate a conversation with the bot by typing or speaking to it. The bot can ask users questions about their interests, level of knowledge, and preferred topics to provide personalized discussions. The bot can also suggest reading materials, videos, and podcasts related to the discussed topics to enhance users' learning experience.

## Protoype
Example code snippet demonstrating how to scrape data from the Stanford Encyclopedia of Philosophy using the BeautifulSoup library. This code extracts the text content of the entry on deontological ethics from the Stanford Encyclopedia of Philosophy. You can modify the url variable to scrape data from different entries. After scraping the data, it needs to be preprocessed by cleaning the text and formatting it for use in NLP models. This involves tasks such as removing stop words, stemming, and tokenisation. There are many libraries available in Python for these tasks, including NLTK, spaCy, and gensim.
```
def main():
import requests
from bs4 import BeautifulSoup

url = 'https://plato.stanford.edu/entries/ethics-deontological/'

response = requests.get(url)
soup = BeautifulSoup(response.content, 'html.parser')

# Extract the article text
article = soup.find('div', {'id': 'main-text-block'}).text

# Print the article
print(article)
main()
```

Example code for training an NLP model using spaCy. This code trains a text classification model using spaCy's built-in textcat component to classify text into four categories: question, philosopher, philosophy, and doubt. It then defines the training data as a list of tuples, where each tuple contains a text and its corresponding labels. Next it adds the labels to the pipeline and defines the training function using spaCy's begin_training method and minibatch function. Finally, it trains the model with 10 epochs and saves it to disk.
```
def main():
import spacy
from spacy.util import minibatch, compounding
import random

# Load the pre-trained model
nlp = spacy.load("en_core_web_sm")

# Define the training data
train_data = [("What is the meaning of life?", {"cats": {"question": True}}),
              ("Socrates was a Greek philosopher", {"cats": {"philosopher": True}}),
              ("Kant's categorical imperative is a central concept in his moral philosophy", {"cats": {"philosophy": True}}),
              ("I am not sure whether free will exists or not", {"cats": {"doubt": True}})]

# Add the labels to the pipeline
if "textcat" not in nlp.pipe_names:
    textcat = nlp.create_pipe("textcat")
    nlp.add_pipe(textcat, last=True)
else:
    textcat = nlp.get_pipe("textcat")

textcat.add_label("question")
textcat.add_label("philosopher")
textcat.add_label("philosophy")
textcat.add_label("doubt")

# Define the training function
def train(model, train_data, epochs):
    optimizer = model.begin_training()
    losses = {}
    for epoch in range(epochs):
        random.shuffle(train_data)
        batches = minibatch(train_data, size=compounding(4.0, 32.0, 1.001))
        for batch in batches:
            texts, annotations = zip(*batch)
            model.update(texts, annotations, sgd=optimizer, drop=0.2, losses=losses)
        print("Epoch: ", epoch, " Losses: ", losses)

# Train the model
train(nlp, train_data, 10)

# Save the trained model
nlp.to_disk("philosophy_nlp_model")
main()
```

Example code using the Rasa framework to build the conversational agent. This code sets up an agent using the trained models, defines a message handler to process user input and generate bot responses, and starts a console-based conversation with the user. You can customize the behavior of the bot by modifying the handle_message function, adding new intents and actions in the Rasa domain file, and retraining the models as needed.
```
def main():
import logging
from rasa.core.agent import Agent
from rasa.core.interpreter import RasaNLUInterpreter
from rasa.core.utils import EndpointConfig
from rasa.core.channels.console import ConsoleInputChannel
from rasa.core.channels.channel import UserMessage
from rasa.core.dispatcher import Dispatcher

# Set up logging
logging.basicConfig(level=logging.INFO)

# Load the trained models
interpreter = RasaNLUInterpreter(model_path="models/nlu")
action_endpoint = EndpointConfig(url="http://localhost:5055/webhook")
agent = Agent.load("models/dialogue", interpreter=interpreter, action_endpoint=action_endpoint)

# Define the message handler
def handle_message(message: UserMessage, dispatcher: Dispatcher):
    # Parse the user message
    parse_data = interpreter.parse(message.text)
    logging.info(f"Parsed user message: {parse_data}")

    # Get the bot response
    response = agent.handle_message(message.text)
    logging.info(f"Bot response: {response}")

    # Send the bot response to the user
    for r in response:
        dispatcher.utter_message(text=r["text"])

# Start the conversation
input_channel = ConsoleInputChannel()
dispatcher = Dispatcher("my-sender-id", input_channel, handle_message)
while True:
    try:
        message = input()
        dispatcher.process_message(message)
    except KeyboardInterrupt:
        logging.info("KeyboardInterrupt detected, stopping conversation.")
        break
main()
```

Example code snippet for implementing machine learning algorithms. This code loads preprocessed data and splits it into training and testing sets. It then trains decision trees, support vector machines, and neural network classifiers on the training data and evaluates their performance on the testing data. Finally, it chooses the best classifier based on its accuracy and prints its name and accuracy. This is just an example and the specific machine learning algorithms and methods will depend on the specific requirements and goals of the finalised Philosophical Discussion Bot.
```
def main():
import numpy as np
import pandas as pd
from sklearn.tree import DecisionTreeClassifier
from sklearn.svm import SVC
from sklearn.neural_network import MLPClassifier
from sklearn.metrics import accuracy_score

# Load preprocessed data
data = pd.read_csv('preprocessed_data.csv')

# Split data into training and testing sets
train_data = data.sample(frac=0.8, random_state=42)
test_data = data.drop(train_data.index)

# Define input and target variables
train_x = train_data.drop(columns=['target'])
train_y = train_data['target']
test_x = test_data.drop(columns=['target'])
test_y = test_data['target']

# Train decision tree classifier
dtc = DecisionTreeClassifier()
dtc.fit(train_x, train_y)
dtc_pred = dtc.predict(test_x)
dtc_acc = accuracy_score(test_y, dtc_pred)

# Train support vector machine classifier
svm = SVC()
svm.fit(train_x, train_y)
svm_pred = svm.predict(test_x)
svm_acc = accuracy_score(test_y, svm_pred)

# Train neural network classifier
mlp = MLPClassifier()
mlp.fit(train_x, train_y)
mlp_pred = mlp.predict(test_x)
mlp_acc = accuracy_score(test_y, mlp_pred)

# Choose the best classifier
classifiers = [('Decision Tree', dtc_acc), ('Support Vector Machine', svm_acc), ('Neural Network', mlp_acc)]
best_classifier = max(classifiers, key=lambda x: x[1])[0]

# Print the best classifier and its accuracy
print(f'The best classifier is {best_classifier} with an accuracy of {max(classifiers, key=lambda x: x[1])[1]}')
main()
```

Example code used as a starting point for deploying the bot on a web application using Flask framework and Heroku cloud platform. This code defines a Flask app with a single endpoint at /message for handling incoming messages. The bot variable is an instance of the ChatbotModel class which generates responses to incoming messages. The handle_message function retrieves the message from the request body, generates a response using the bot variable, and returns the response as a JSON object. Finally, the if __name__ == '__main__' block runs the Flask app. To deploy this app on Heroku, you can create a requirements.txt file with all the dependencies required by your app and a Procfile file that specifies the command to run the app. You can then push the code to a Heroku remote repository and start the app. For more details on deploying Flask apps on Heroku, you can refer to the Heroku documentation: https://devcenter.heroku.com/categories/reference
```
def main():
from flask import Flask, request, jsonify
import chatbot_model # your chatbot model module

app = Flask(__name__)

# initialize chatbot model
bot = chatbot_model.ChatbotModel()

# define endpoint for handling incoming messages
@app.route('/message', methods=['POST'])
def handle_message():
    # get message from request body
    message = request.json['message']
    # generate response using chatbot model
    response = bot.generate_response(message)
    # return response as JSON object
    return jsonify({'response': response})

if __name__ == '__main__':
    # run Flask app
    app.run()
main()
```

## Demo
Here's a simple demo of how the Philosophical Discussion Bot works:

**User:** Hi, I'm interested in learning about ethics.

**Bot:** Great! Ethics is a branch of philosophy that deals with moral principles and values. What would you like to know about ethics?

**User:** Can you explain the difference between deontology and consequentialism?

**Bot:** Sure! Deontology is a moral theory that emphasizes the importance of following moral rules and duties regardless of their consequences. Consequentialism, on the other hand, is a moral theory that emphasizes the importance of the outcomes or consequences of actions in determining their morality. Would you like to discuss more about these theories?

**User:** Yes, I'm still confused about how to apply them in real-life situations.

**Bot:** That's a great question! The application of deontology and consequentialism can be challenging in some situations, as they may conflict with each other. For example, what would you do if you had to choose between lying to save a person's life or telling the truth and risking their life? Would you like to discuss this further?

**User:** Yes, I would like to know more about the strengths and weaknesses of these theories.

**Bot:** Sure! Deontology has the advantage of providing clear and consistent moral rules that apply to all situations, but it can also lead to inflexibility and moral absolutism. Consequentialism has the advantage of promoting the best outcomes for the greatest number of people, but it can also lead to moral relativism and justify immoral actions if they produce good results. Would you like to read more about these theories?

**User:** Yes, please recommend some articles or books about ethics.

**Bot:** Sure! Here are some articles and books about ethics that you might find interesting:
- "The Nicomachean Ethics" by Aristotle
- "Groundwork of the Metaphysics of Morals" by Immanuel Kant
- "Utilitarianism" by John Stuart Mill
- "An Enquiry Concerning the Principles of Morals" by David Hume
- "Justice: What's the Right Thing to Do?" by Michael Sandel

Thank you for the discussion! Let me know if you have any other questions or comments.

## Data sources and AI methods
The bot uses natural language processing (NLP) and machine learning techniques to understand and respond to users' inputs. The bot's knowledge base includes various philosophical concepts, theories, and arguments obtained from online sources, such as Stanford Encyclopedia of Philosophy and Philosophy Now magazine. The bot's responses are generated using a combination of rule-based and machine learning-based approaches.

A starting point of sources and resources required to build the Philosophical Language Bot include:
* Stanford Encyclopedia of Philosophy: a comprehensive online encyclopedia of philosophical concepts and topics.
* Philosophy Now magazine: a popular magazine that covers a wide range of philosophical topics and issues.
* JSTOR: a digital library that provides access to academic journals, books, and primary sources in various fields, including philosophy.
* Internet Encyclopedia of Philosophy: an online encyclopedia of philosophy that covers a wide range of philosophical topics and issues.
* Philosophical journals: various academic journals in the field of philosophy, such as Philosophy of Science and Philosophical Quarterly.
* PhilPapers: an online research database of academic philosophy articles, books, and reviews.
* Open access repositories: such as arXiv, which hosts research papers in various fields, including philosophy.
* Programming languages: Python, Java, or JavaScript.
* Natural Language Processing (NLP) frameworks such as spaCy, NLTK, or Stanford CoreNLP for data preprocessing and building NLP models.
* Conversational AI frameworks such as Rasa, Dialogflow, or IBM Watson Assistant to build the conversational agent.
* Cloud hosting services such as AWS or Azure to deploy the bot.
* Version control systems such as Git to manage and track changes in the codebase.
* Software development methodologies such as Agile or Scrum to manage the project effectively.
* API documentation for third-party services used in the project, such as the Stanford Encyclopedia of Philosophy API or Google Books API.
* Web development frameworks such as Flask or Django to build the web application for the bot.
* Database management systems such as MySQL or MongoDB to store and manage the bot's data.
* Encryption and authentication methods such as Transport Layer Security (TLS),Secure Sockets Layer (SSL), OAuth, JSON Web Tokens (JWT), HTTPS, and/or Two-Factor Authentication (2FA).

## Challenges
The project has some limitations and ethical considerations that need to be addressed. One challenge is ensuring that the bot's responses are accurate and unbiased, as philosophical concepts and theories can have multiple interpretations and perspectives. Another challenge is maintaining the user's privacy and data security, as the bot may collect personal information during the conversation. 

**Natural Language Processing**
The bot needs to be able to understand and process users' inputs in natural language. The natural language processing (NLP) component can be built using a deep learning model, such as a recurrent neural network (RNN) or transformer-based model like BERT. The NLP model can be trained on a corpus of philosophical texts and conversations to learn to recognize and understand different philosophical concepts and topics.

**Knowledge Representation**
The bot needs to have a knowledge representation system that stores and organizes information about philosophical concepts, theories, and arguments. The knowledge base can be built using a semantic web technology, such as RDF (Resource Description Framework) or OWL (Web Ontology Language). The knowledge base can be populated with information from various sources, such as Stanford Encyclopedia of Philosophy, Philosophy Now magazine, and other online philosophy resources.

**Dialogue System**
The bot needs to have a dialogue system that generates responses to users' inputs based on the knowledge representation system. The dialogue system can be built using a rule-based or machine learning-based approach, or a combination of both. The rule-based approach can use predefined templates or patterns to generate responses based on the user's input, while the machine learning-based approach can use a neural network to generate responses based on the context and previous dialogue history.

**User Interface**
The bot needs a user interface that allows users to interact with it through text or speech. The user interface can be a web or mobile application that sends users' inputs to the NLP component and displays the bot's responses generated by the dialogue system. The user interface can also include features such as personalized discussion topics, recommended reading materials, and interactive learning tools to enhance users' learning experience.

**Deployment**
The bot can be deployed on a cloud platform such as AWS, Azure, or Google Cloud. The deployment process involves creating a virtual machine or container with the necessary dependencies and libraries, such as Python, TensorFlow, and Flask. The deployment also involves configuring the server and network settings to handle incoming requests from the user interface and sending responses back to the user.

**Testing and Evaluation**
The bot needs to be tested and evaluated to ensure its accuracy and effectiveness in engaging users in philosophical discussions. The testing and evaluation can be done using various metrics, such as precision, recall, F1-score, and user satisfaction rating. The bot can be evaluated using a test set of philosophical texts and conversations and a group of human evaluators who can provide feedback on the bot's performance and usability.

## What next?
To create a working prototype of the Philosophical Discussion Bot, we will need to use natural language processing (NLP) and machine learning techniques. Example code for each step can be found above under **Protoype**. Here's a basic outline of the steps involved in building the bot:

**Collect and preprocess data:** We will need to collect and preprocess data on various philosophical concepts, theories, and arguments from online sources, such as Stanford Encyclopedia of Philosophy and Philosophy Now magazine. The data will need to be cleaned and formatted for use in our NLP models.

**Train NLP models:** We will use NLP models, such as sentiment analysis, named entity recognition, and topic modeling, to analyze the collected data and extract useful information. We can use tools like spaCy or NLTK to train and test these models.

**Build a conversational agent:** We will use a conversational agent framework, such as Rasa or Dialogflow, to build the bot's conversational flow. The bot will need to be able to recognize user input, generate appropriate responses based on the input, and provide recommendations for reading materials and other resources.

**Implement machine learning algorithms:** We will use machine learning algorithms, such as decision trees, support vector machines, and neural networks, to improve the accuracy of the bot's responses. These algorithms will help the bot learn from user input and improve its performance over time.

**Deploy the bot:** Once the bot is built and tested, we can deploy it on a web or mobile application using a cloud hosting service, such as AWS or Azure. We will need to ensure that the bot is secure, reliable, and scalable.

The Philosophical Discussion Bot has great potential for growth and further development. Some ways to expand the project could include:
* Integrating more advanced AI techniques: The current version of the bot uses a combination of rule-based and machine learning-based approaches to generate responses. To improve the accuracy and effectiveness of the bot's responses, more advanced AI techniques could be integrated, such as deep learning, reinforcement learning, or unsupervised learning.
* Incorporating more philosophical topics: The current version of the bot covers a variety of philosophical concepts and theories, but there are many more topics that could be included, such as political philosophy, philosophy of religion, and aesthetics. Adding more topics would make the bot more comprehensive and engaging.
* Improving user engagement: The current version of the bot provides personalized discussions based on users' interests and knowledge level, but there are other ways to improve user engagement. For example, the bot could include interactive quizzes, games, and simulations to make learning about philosophy more fun and engaging.

To move on with these improvements, further assistance from experts in AI, philosophy, and user experience design could be helpful. Collaboration with philosophers and researchers in the field could also provide valuable insights and feedback for the development of the project. Additionally, user feedback and testing could be conducted to identify areas for improvement and ensure that the bot meets the needs of its users.

## Acknowledgments
This project drew inspiration from various sources, including:

* "The Philosophy Book: Big Ideas Simply Explained" by Will Buckingham, Douglas Burnham, and Peter J. King, which provided a comprehensive overview of various philosophical concepts and theories in an accessible way.
* "The Story of Philosophy" by Will Durant and "The Consolations of Philosophy" by Alain de Botton, which offered insightful perspectives on the history and practical applications of philosophy.
* "Philosophy Now", a website that publishes articles, interviews, and reviews about philosophy for a general audience.
* The course "Introduction to Philosophy" by the University of Edinburgh on Coursera, which provided a basic understanding of philosophical concepts and theories.
* The AI language models developed by OpenAI, which provided powerful natural language processing capabilities for building conversational bots.
