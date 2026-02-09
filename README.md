# ![aws](https://github.com/julien-muke/Search-Engine-Website-using-AWS/assets/110755734/01cd6124-8014-4baa-a5fe-bd227844d263) Serverless RAG AI Chatbot on AWS 🧠

<div align="center">

  <br />
    <a href="#" target="_blank">
      <img src="https://github.com/user-attachments/assets/fc83d541-0fff-4d40-bea5-82340397a251" alt="Project Banner">
    </a>
  <br />

<h3 align="center">Serverless RAG AI Chatbot on AWS using Amazon Bedrock</h3>

   <div align="center">
     Build this hands-on demo step by step with my detailed tutorial on <a href="http://www.youtube.com/@julienmuke/videos" target="_blank"><b>Julien Muke</b></a> YouTube. Feel free to subscribe 🔔!
    </div>
</div>

## <a name="introduction">🤖 Introduction</a>

Welcome to part 02 of the Serverless AI Chatbot series! In this tutorial, we evolve our basic chatbot into a powerful Enterprise Knowledge Assistant. By implementing Retrieval-Augmented Generation (RAG), we enable the AI to answer questions based on your private documents (PDFs, FAQs, and Policies) with pinpoint accuracy and citations.

👉 See Part 01 (Foundations): [https://youtu.be/63McfqGULvA]

## 🔎 Overview: Why RAG?

In todays tech field, generic AI is "good," but Grounded AI is better. RAG solves the two biggest problems in AI:

1. Hallucinations: The AI only answers based on provided facts.

2. Static Knowledge: You don't need to retrain the model; just update your documents, and the AI "knows" instantly.

## 🛠 Tech Stack

- Amazon Bedrock Knowledge Bases: The orchestration engine.

- Amazon OpenSearch Serverless: Our high-performance Vector Database.

- Anthropic Claude 3 Haiku: The LLM for fast, cost-effective reasoning.

- AWS Lambda: Backend logic using the RetrieveAndGenerate API.

- Amazon S3: Document storage and static web hosting.

## 🏗 Architecture Diagram

1. Admin uploads a PDF to the S3 Source Bucket.

2. Bedrock Knowledge Base syncs the data, converts it to embeddings (Titan v2), and stores it in OpenSearch Serverless.

3. User asks a question via the S3-hosted UI.

4. Lambda triggers the RetrieveAndGenerate API.

5. Bedrock finds the relevant context, feeds it to Claude 3, and returns a grounded answer.

## ➡️ Step 1: Create the Knowledge Source (S3)

1. Create a new S3 bucket named `your-name-kb-source`

2. Upload a sample document e.g.: 

- Company_Policy_2025.pdf
- Product_docs.pdf
- HR_manuals.pdf
- IT_troubleshooting_guides.pdf

3. Note: Ensure this bucket is in the same region as your Bedrock Knowledge Base.

## ➡️ Step 2: Setup Bedrock Knowledge Base

1. Go to Amazon Bedrock Console

2. On the left menu, click: Knowledge Bases → Create Knowledge Base

3. Give it a name: `your-name-rag-knowledge`

4. Data Source: Select the S3 bucket you created in Step 1.

5. Embeddings Model: Choose Titan Text Embeddings v2.

6. For the vector database: Select "Quick Create a new vector store". AWS will automatically provision Amazon OpenSearch Serverless for you.

This is an AWS-managed option (no infrastructure, fully managed, clean and scalable).

7. Click Create & Sync and wait for the status to become Active.

8. Once active, click the Sync button in the Data Source section.

AWS is now chunking your documents, generating embeddings, and storing vectors.

## ➡️ Step 3: Update the Backend (Lambda)

We are moving away from `invoke_model` to the more advanced `retrieve_and_generate` API. This handles the retrieval and the LLM prompting in one secure step.

`lambda_function.py`

```python
import boto3
import json
import os

bedrock_agent_runtime = boto3.client('bedrock-agent-runtime')

def lambda_handler(event, context):
    # 1. Parse user input
    body = json.loads(event['body'])
    user_query = body['message']
    
    # 2. Configuration (Replace with your IDs)
    kb_id = "YOUR_KB_ID" 
    model_arn = "arn:aws:bedrock:us-east-1::foundation-model/anthropic.claude-3-haiku-20240307-v1:0"

    # 3. RAG API Call
    response = bedrock_agent_runtime.retrieve_and_generate(
        input={'text': user_query},
        retrieveAndGenerateConfiguration={
            'type': 'KNOWLEDGE_BASE',
            'knowledgeBaseConfiguration': {
                'knowledgeBaseId': kb_id,
                'modelArn': model_arn
            }
        }
    )

    return {
        'statusCode': 200,
        'headers': {
            'Access-Control-Allow-Origin': '*',
            'Content-Type': 'application/json'
        },
        'body': json.dumps({'message': response['output']['text']})
    }
```

## ➡️ Step 4: IAM Permissions

Your Lambda role needs permission to talk to the Bedrock Agent. Attach this inline policy to your Lambda execution role:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "bedrock:RetrieveAndGenerate",
            "Resource": "*"
        }
    ]
}
```

## ➡️ Step 5: API + Frontend

We don’t need to redesign anything here, we’ll keep the same API Gateway, we’ll keep the same frontend hosted on S3.

The only difference is now our chatbot is knowledge-aware and enterprise-ready.

## ➡️ Step 6: Testing

You can use the exact same Frontend (HTML/CSS) from [Part 01](https://youtu.be/63McfqGULvA). Simply update the API Gateway URL in your `index.html` (if it changed).

Now let’s test this, I uploaded a real Company Internal Policy document earlier.

1. Ask question like: "What is the company’s data privacy policy?"

2. The AI will search the PDF in S3 and provide an answer.

3. Pro-Tip: If you add a new PDF to S3 and hit Sync in Bedrock, the chatbot will have that knowledge instantly!

## 💼 WHY THIS MATTERS IN THE REAL WORLD

Organizations are using this exact pattern today for:

- Internal knowledge assistants.
- HR helpdesk chatbots.
- IT support assistants.
- Compliance and policy bots.
- Financial and insurance document assistants.

## 🗑️ Clean Up

To avoid unexpected AWS charges:

2. Delete the OpenSearch Serverless Collection (This is the most expensive part).

3. Delete the Bedrock Knowledge Base.

4. Empty and delete the S3 Buckets.

5. Delete the Lambda and API Gateway.

## ✨ Conclusion

By completing Part 02, you've built a production-ready RAG pipeline. This architecture is used by major corporations to build internal search engines, customer support bots, and research assistants.

If this project helped you, please give it a ⭐ and subscribe to [My YouTube Channel](http://www.youtube.com/@julienmuke/videos)!


<a href="http://www.youtube.com/@julienmuke/videos" target="_blank">
      <img src="https://github.com/user-attachments/assets/a864c284-5647-4f4d-af70-386d7e0efadc" alt="Project Banner">
</a>