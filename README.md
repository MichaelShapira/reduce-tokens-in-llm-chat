# Reduce the number of tokens when using LLM for chat 

One of the most common use cases for LLM is chat. However, LLM is stateless and doesn't have a memory to store the chat conversation.

Given this fact, we need to send to LLM all the previous history of the chat so it will understand the content and response properly. 

This may be challenging from a cost perspective since we need to send to LLM the same information that LLM has already answered. Most LLM providers expose their models through APIs and change input and output tokens. It means that with each follow-up question in chat, you pay extra for the questions that were already answered.

## Current Architecture

Usually, the process looks like this: The user sends a question to chat. Chat history is being stored in a low-latency database (like DynamoDB), and each time the whole history + new question is sent to LLM. The response is being added to the database storage and also sent to the user.


![image.png](attachment:15419b2f-1d8f-4143-b5e4-5340fb5186dd.png)


## Approach to reduce the number of tokens

If my chat history is only two or three questions, I probably should not do anything. Tokens overhead are not so large.

But if my conversation contains multiple questions, I propose the following enhancement:

After the three or four questions/answer pairs, don't store the conversation. Summarize the conversation to this point, and store only the summarized text. All subsequent requests are made in the context of the summarized text.

The architecture remains the same. But now we sent to LLM a summarized conversation and user question. In response, we ask that you provide a new summary that includes the latest question and an answer to the user's question.
