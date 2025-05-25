# LocalTechNews-AI
LocalTechNews-AI is a fully self-hosted, local-first automation workflow that builds and sends a summarized tech newsletter straight to your inbox — powered entirely by tools running on your own machine.


his project combines:

🧠 Local AI via LM Studio (using models like Mistral, LLaMA, Phi, etc.)

🔧 n8n for workflow orchestration

🐳 Docker for portable deployment

✉️ Gmail integration for delivering clean, readable digests

And best of all?

✅ No OpenAI
✅ No external APIs
✅ No cloud dependencies

Everything runs locally and is completely privacy-respecting.

⚙️ How It Works:
Trigger: Start the automation manually or on a schedule using n8n.

Input Source: Manually define your tech interests or plug in your own content-fetching logic.

Local LLM (via LM Studio): Summarize the content using a fully offline language model.

Email Formatter: Cleanly convert responses to an HTML format for reading.

Gmail Node: Sends the newsletter straight to your inbox 

### Demo Screenshot

![image](https://github.com/user-attachments/assets/c978660d-298b-4810-a05f-c3b031b3daa5)

## ------->

![image](https://github.com/user-attachments/assets/3a8b0e8d-3b4a-49fd-ae01-2912a0dcbccc)

# ------->

![image](https://github.com/user-attachments/assets/a91a4b86-18cc-4f0a-83ca-4d9d26740c36)


 ### Local Setup

 - Run n8n via Docker

2. Run LM Studio
- Download and install LM Studio

- Load a local model (e.g., Mistral 7B, LLaMA, Phi-2)

 - Enable the OpenAI-compatible server in LM Studio (usually on http://localhost:5678)

- Test using curl or Postman to ensure LM Studio is serving responses.

📩 3. Configure Gmail Node
In your n8n workflow, set up:

 - Gmail OAuth2 or SMTP credentials( might be overwhelming but i can help )

- A destination email (yourself or a list)

 ### Privacy & Control
This setup is designed for:

- Developers & tinkerers

 - Indie founders

- Privacy advocates

## WOrkflow  

- copy and  save as .json paste it your n8n 

``` json
{
  "nodes": [
    {
      "parameters": {
        "model": {
          "__rl": true,
          "value": "llama-3.2-1b-instruct",
          "mode": "list",
          "cachedResultName": "llama-3.2-1b-instruct"
        },
        "options": {}
      },
      "id": "b5042427-1b1a-4477-8707-bbe36bef0c4e",
      "name": "OpenAI Chat Model",
      "type": "@n8n/n8n-nodes-langchain.lmChatOpenAi",
      "position": [
        860,
        900
      ],
      "typeVersion": 1.2,
      "credentials": {
        "openAiApi": {
          "id": "s2WCiIktcsnTwMuA",
          "name": "llama-3.2-1b-instruct"
        }
      }
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=Summarize the most relevant tech news published in the past 7 days. Today is {{ $now }}",
        "options": {
          "systemMessage": "=Write in clear, concise, and easy-to-understand English, as if curating a weekly tech newsletter.\n\nPrioritize stories related to the following topics: {{ $json.Interests }}.\nIf those topics aren't present, choose the most impactful, widely reported, and noteworthy tech events from the week.\n\nFor each news item:\n\nProvide a short, informative summary (1–3 sentences max)\n\nInclude the article title and a direct link to the original source\n\nAvoid duplicate stories.\n\nInclude exactly {{ $json['Number of news items to include'] }} distinct news items.\nMaintain a tone that's friendly yet professional, suitable for a broad audience of tech enthusiasts."
        }
      },
      "id": "5d563a2e-43c0-43f6-9c8f-1ae951447e65",
      "name": "News reader AI",
      "type": "@n8n/n8n-nodes-langchain.agent",
      "position": [
        880,
        680
      ],
      "typeVersion": 1.9
    },
    {
      "parameters": {
        "sendTo": "your mail id",
        "subject": "Weekly tech newsletter",
        "message": "={{ $json.data }}",
        "options": {}
      },
      "id": "9b26da05-dcd8-4ad8-a06d-e1c73b315ba0",
      "name": "Send Newsletter",
      "type": "n8n-nodes-base.gmail",
      "position": [
        1480,
        740
      ],
      "webhookId": "0de8b6e8-8611-48a9-ba25-1d023698f577",
      "typeVersion": 2.1,
      "credentials": {
        "gmailOAuth2": {
          "id": "drpqPBgFHTjoPkRi",
          "name": "Gmail account"
        }
      }
    },
    {
      "parameters": {
        "mode": "markdownToHtml",
        "markdown": "={{ $json.output }}",
        "options": {}
      },
      "id": "ffbd42d4-8e15-49d6-909c-8b0503412090",
      "name": "Convert Response to an Email-Friendly Format",
      "type": "n8n-nodes-base.markdown",
      "position": [
        1240,
        680
      ],
      "typeVersion": 1
    },
    {
      "parameters": {
        "content": "topics\n",
        "height": 240,
        "width": 220,
        "color": 3
      },
      "id": "afd2f5f9-842d-439f-904a-14025ddd35df",
      "name": "Sticky Note3",
      "type": "n8n-nodes-base.stickyNote",
      "position": [
        580,
        680
      ],
      "typeVersion": 1
    },
    {
      "parameters": {
        "mode": "retrieve-as-tool",
        "toolName": "get_news",
        "toolDescription": "Call this tool to get the latest news articles.",
        "memoryKey": "news_store_key",
        "topK": 20
      },
      "id": "0ce2fcef-4e79-4d2c-9977-a4405a1cc5e8",
      "name": "Get News Articles",
      "type": "@n8n/n8n-nodes-langchain.vectorStoreInMemory",
      "position": [
        1000,
        880
      ],
      "typeVersion": 1.1
    },
    {
      "parameters": {
        "model": "text-embedding-nomic-embed-text-v1.5",
        "options": {}
      },
      "id": "d099ead2-b030-47a4-907a-582a457110e9",
      "name": "Embeddings OpenAI2",
      "type": "@n8n/n8n-nodes-langchain.embeddingsOpenAi",
      "position": [
        1200,
        1000
      ],
      "typeVersion": 1.2,
      "credentials": {
        "openAiApi": {
          "id": "s2WCiIktcsnTwMuA",
          "name": "llama-3.2-1b-instruct"
        }
      }
    },
    {
      "parameters": {
        "content": "### email:",
        "height": 240,
        "width": 220,
        "color": 3
      },
      "id": "c4537193-dcd6-4bdf-be24-4010e75ea103",
      "name": "Sticky Note",
      "type": "n8n-nodes-base.stickyNote",
      "position": [
        1420,
        680
      ],
      "typeVersion": 1
    },
    {
      "parameters": {
        "assignments": {
          "assignments": [
            {
              "id": "018882ca-37c3-45af-944f-2081b0605065",
              "name": "Interests",
              "type": "string",
              "value": "AI, games, gadgets"
            },
            {
              "id": "4cfdafc1-47a4-41cc-9eb8-72880ea34511",
              "name": "Number of news items to include",
              "type": "string",
              "value": "20"
            }
          ]
        },
        "options": {}
      },
      "id": "01faf51a-10e7-4738-85f2-011bc73d3143",
      "name": "topics of interest",
      "type": "n8n-nodes-base.set",
      "position": [
        640,
        700
      ],
      "typeVersion": 3.4
    },
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "field": "weeks",
              "triggerAtDay": [
                1
              ],
              "triggerAtHour": 5
            }
          ]
        }
      },
      "id": "dc7d2fcf-9f10-4204-ab16-a1a877bc356d",
      "name": "trigger",
      "type": "n8n-nodes-base.scheduleTrigger",
      "position": [
        420,
        740
      ],
      "typeVersion": 1.2
    }
  ],
  "connections": {
    "OpenAI Chat Model": {
      "ai_languageModel": [
        [
          {
            "node": "News reader AI",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "News reader AI": {
      "main": [
        [
          {
            "node": "Convert Response to an Email-Friendly Format",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Convert Response to an Email-Friendly Format": {
      "main": [
        [
          {
            "node": "Send Newsletter",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Get News Articles": {
      "ai_tool": [
        [
          {
            "node": "News reader AI",
            "type": "ai_tool",
            "index": 0
          }
        ]
      ]
    },
    "Embeddings OpenAI2": {
      "ai_embedding": [
        [
          {
            "node": "Get News Articles",
            "type": "ai_embedding",
            "index": 0
          }
        ]
      ]
    },
    "topics of interest": {
      "main": [
        [
          {
            "node": "News reader AI",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "trigger": {
      "main": [
        [
          {
            "node": "topics of interest",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "pinData": {},
  "meta": {
    "templateCredsSetupCompleted": true,
    "instanceId": "d581174f09558f6883b4b40f81637880e7d8f0dc8f220c3d54c52bd28ee10bbc"
  }
}
```

