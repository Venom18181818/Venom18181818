- 👋 Hi, I’m @Venom18181818
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m lookin!---
Venom18181818/Venom18181818 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->




import os
import json
import pandas as pd
from textblob import TextBlob
import matplotlib.pyplot as plt

# Path to the folder containing JSON files
data_folder = "/path/to/your/data/folder"

# List to store conversation data
conversations = []

# Read all JSON files in the folder
for file_name in os.listdir(data_folder):
    if file_name.endswith(".json"):
        file_path = os.path.join(data_folder, file_name)
        with open(file_path, 'r', encoding='utf-8') as file:
            data = json.load(file)
            conversations.append(data)

# Process conversations
conversation_data = []
for idx, conversation in enumerate(conversations, start=1):
    customer_text = " ".join([entry["customer"] for entry in conversation if "customer" in entry])
    agent_text = " ".join([entry["agent"] for entry in conversation if "agent" in entry])
    full_text = customer_text + " " + agent_text

    # Sentiment analysis using TextBlob
    sentiment = TextBlob(full_text).sentiment
    polarity = sentiment.polarity
    subjectivity = sentiment.subjectivity

    # Determine sentiment label
    if polarity > 0:
        sentiment_label = "Positive"
    elif polarity < 0:
        sentiment_label = "Negative"
    else:
        sentiment_label = "Neutral"

    # Append data
    conversation_data.append({
        "Serial Number": idx,
        "Customer Text": customer_text,
        "Agent Text": agent_text,
        "Polarity": polarity,
        "Subjectivity": subjectivity,
        "Sentiment": sentiment_label
    })

# Create a DataFrame
df = pd.DataFrame(conversation_data)

# Calculate overall satisfaction level
positive_count = df[df["Sentiment"] == "Positive"].shape[0]
negative_count = df[df["Sentiment"] == "Negative"].shape[0]
neutral_count = df[df["Sentiment"] == "Neutral"].shape[0]

# Satisfaction statistics
total_conversations = len(df)
satisfaction_level = (positive_count / total_conversations) * 100

# Print EDA summary
print(f"Total Conversations: {total_conversations}")
print(f"Positive Conversations: {positive_count}")
print(f"Negative Conversations: {negative_count}")
print(f"Neutral Conversations: {neutral_count}")
print(f"Overall Satisfaction Level: {satisfaction_level:.2f}%")

# Visualization
plt.figure(figsize=(8, 6))
df["Sentiment"].value_counts().plot(kind="bar", color=["green", "red", "blue"])
plt.title("Sentiment Analysis of Conversations")
plt.xlabel("Sentiment")
plt.ylabel("Number of Conversations")
plt.show()



