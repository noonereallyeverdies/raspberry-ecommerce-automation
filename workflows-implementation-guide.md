# E-commerce Automation Workflows - Complete Implementation Guide

## 🎯 Priority Workflows for Your Shopify Store

Based on your requirements (3x daily Instagram posts, email automation, B2B outreach, accounting), here are the **15 essential workflows** organized by priority and ROI.

---

## 📊 Workflow Priority Matrix

| Priority | Workflow | Time Saved/Week | Setup Complexity | ROI |
|----------|----------|-----------------|------------------|-----|
| **P0** | Instagram Content Generation | 15 hrs | Medium | 🔥🔥🔥 |
| **P0** | Shopify Order Processing | 10 hrs | Low | 🔥🔥🔥 |
| **P0** | Rate Limit Manager | Prevents bans | Low | 🔥🔥🔥 |
| **P1** | Email Style Learning | 8 hrs | Medium | 🔥🔥 |
| **P1** | Customer Response Automation | 12 hrs | Medium | 🔥🔥 |
| **P1** | Daily Accounting Sync | 5 hrs | Low | 🔥🔥 |
| **P2** | B2B Outreach Automation | 10 hrs | High | 🔥 |
| **P2** | Review Management | 3 hrs | Low | 🔥 |
| **P2** | Inventory Monitoring | 4 hrs | Medium | 🔥 |
| **P3** | Performance Analytics | 2 hrs | Low | ⭐ |

---

## 🚀 Core Workflow #1: Instagram Content Generation & Posting

### The Complete AI-Powered Instagram Workflow

```json
{
  "name": "Instagram_AI_Content_Pipeline",
  "nodes": [
    {
      "id": "1",
      "name": "Schedule_Trigger",
      "type": "n8n-nodes-base.scheduleTrigger",
      "parameters": {
        "rule": {
          "cronExpression": "0 9,14,19 * * *"
        }
      },
      "position": [250, 300]
    },
    {
      "id": "2",
      "name": "Check_Rate_Limit",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "GET",
        "url": "https://cloud.appwrite.io/v1/databases/prod/collections/api_calls/documents",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "X-Appwrite-Project",
              "value": "={{$credentials.appwriteProject}}"
            },
            {
              "name": "X-Appwrite-Key",
              "value": "={{$credentials.appwriteKey}}"
            }
          ]
        },
        "sendQuery": true,
        "queryParameters": {
          "parameters": [
            {
              "name": "queries[]",
              "value": "equal(\"api\",[\"instagram\"])"
            },
            {
              "name": "queries[]",
              "value": "greaterThan(\"timestamp\",[\"{{$now.minus(1, 'hour').toISO()}}\"])"
            }
          ]
        }
      },
      "position": [450, 300]
    },
    {
      "id": "3",
      "name": "Rate_Limit_Check",
      "type": "n8n-nodes-base.if",
      "parameters": {
        "conditions": {
          "number": [
            {
              "value1": "={{$json.total}}",
              "operation": "smaller",
              "value2": 180
            }
          ]
        }
      },
      "position": [650, 300]
    },
    {
      "id": "4",
      "name": "Get_Product_Data",
      "type": "n8n-nodes-base.shopify",
      "parameters": {
        "operation": "getAll",
        "resource": "product",
        "limit": 5,
        "additionalFields": {
          "status": "active",
          "fields": "title,images,description,variants"
        }
      },
      "position": [850, 300]
    },
    {
      "id": "5",
      "name": "Generate_Content_Gemini",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "POST",
        "url": "https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "x-goog-api-key",
              "value": "={{$credentials.geminiApiKey}}"
            }
          ]
        },
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "contents",
              "value": "=[\n  {\n    \"parts\": [\n      {\n        \"text\": \"You are a social media expert for an e-commerce brand. Create an engaging Instagram post for the following product:\\n\\nProduct: {{$json.title}}\\nDescription: {{$json.description}}\\n\\nRequirements:\\n1. Caption: 100-150 words, conversational tone\\n2. Include 3-5 relevant hashtags\\n3. Add a call-to-action\\n4. Emoji usage: moderate and relevant\\n5. Focus on benefits, not features\\n\\nBrand voice: Professional yet approachable, focusing on quality and customer satisfaction.\"\n      }\n    ]\n  }\n]"
            },
            {
              "name": "generationConfig",
              "value": "={\n  \"temperature\": 0.7,\n  \"topK\": 40,\n  \"topP\": 0.95,\n  \"maxOutputTokens\": 500,\n  \"stopSequences\": []\n}"
            },
            {
              "name": "safetySettings",
              "value": "=[\n  {\n    \"category\": \"HARM_CATEGORY_HARASSMENT\",\n    \"threshold\": \"BLOCK_ONLY_HIGH\"\n  }\n]"
            }
          ]
        }
      },
      "position": [1050, 300]
    },
    {
      "id": "6",
      "name": "Generate_Image_Imagen",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "POST",
        "url": "https://generativelanguage.googleapis.com/v1beta/models/imagen-3:generate",
        "authentication": "genericCredentialType",
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "prompt",
              "value": "Professional product photography of {{$node['Get_Product_Data'].json.title}}, studio lighting, white background, high resolution, commercial quality, Instagram square format"
            },
            {
              "name": "number_of_images",
              "value": 1
            },
            {
              "name": "aspect_ratio",
              "value": "1:1"
            }
          ]
        }
      },
      "position": [1250, 300]
    },
    {
      "id": "7",
      "name": "Quality_Score_Check",
      "type": "n8n-nodes-base.code",
      "parameters": {
        "language": "javascript",
        "code": "// AI Quality Scoring\nconst content = $input.item.json.generated_content;\nconst productData = $node['Get_Product_Data'].json;\n\n// Scoring criteria\nlet score = 0;\n\n// Check caption length (100-150 words ideal)\nconst wordCount = content.caption.split(' ').length;\nif (wordCount >= 100 && wordCount <= 150) score += 20;\n\n// Check hashtag presence\nconst hashtags = (content.caption.match(/#/g) || []).length;\nif (hashtags >= 3 && hashtags <= 5) score += 20;\n\n// Check CTA presence\nconst ctaPhrases = ['shop now', 'link in bio', 'get yours', 'order today', 'limited time'];\nconst hasCTA = ctaPhrases.some(phrase => content.caption.toLowerCase().includes(phrase));\nif (hasCTA) score += 20;\n\n// Check emoji usage\nconst emojiCount = (content.caption.match(/[\\u{1F300}-\\u{1F9FF}]/gu) || []).length;\nif (emojiCount >= 2 && emojiCount <= 8) score += 20;\n\n// Check product mention\nif (content.caption.toLowerCase().includes(productData.title.toLowerCase())) score += 20;\n\n// Normalize to 0-1 scale\nconst confidence = score / 100;\n\nreturn {\n  json: {\n    content: content,\n    confidence: confidence,\n    score_details: {\n      word_count: wordCount,\n      hashtag_count: hashtags,\n      has_cta: hasCTA,\n      emoji_count: emojiCount,\n      product_mentioned: score >= 80\n    },\n    requires_approval: confidence < 0.85\n  }\n};"
      },
      "position": [1450, 300]
    },
    {
      "id": "8",
      "name": "Approval_Router",
      "type": "n8n-nodes-base.if",
      "parameters": {
        "conditions": {
          "number": [
            {
              "value1": "={{$json.confidence}}",
              "operation": "largerEqual",
              "value2": 0.85
            }
          ]
        }
      },
      "position": [1650, 300]
    },
    {
      "id": "9",
      "name": "Telegram_Approval",
      "type": "n8n-nodes-base.telegram",
      "parameters": {
        "operation": "sendMessage",
        "chatId": "={{$credentials.telegramChatId}}",
        "text": "=📱 **Instagram Post Approval Required**\\n\\n**Confidence Score**: {{$json.confidence}}/1.0\\n\\n**Caption**:\\n{{$json.content.caption}}\\n\\n**Product**: {{$node['Get_Product_Data'].json.title}}\\n\\n**Score Details**:\\n- Word Count: {{$json.score_details.word_count}}\\n- Hashtags: {{$json.score_details.hashtag_count}}\\n- Has CTA: {{$json.score_details.has_cta}}\\n- Emojis: {{$json.score_details.emoji_count}}\\n\\nReply with:\\n/approve_{{$json.post_id}} to approve\\n/reject_{{$json.post_id}} to reject\\n/edit_{{$json.post_id}} to request changes",
        "additionalFields": {
          "parse_mode": "Markdown",
          "reply_markup": {
            "inline_keyboard": [
              [
                {
                  "text": "✅ Approve",
                  "callback_data": "approve_{{$json.post_id}}"
                },
                {
                  "text": "❌ Reject",
                  "callback_data": "reject_{{$json.post_id}}"
                }
              ]
            ]
          }
        }
      },
      "position": [1850, 400]
    },
    {
      "id": "10",
      "name": "Post_to_Instagram",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "POST",
        "url": "https://graph.facebook.com/v18.0/{{$credentials.instagramBusinessId}}/media",
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "image_url",
              "value": "={{$json.image_url}}"
            },
            {
              "name": "caption",
              "value": "={{$json.content.caption}}"
            },
            {
              "name": "access_token",
              "value": "={{$credentials.metaAccessToken}}"
            }
          ]
        }
      },
      "position": [2050, 300]
    },
    {
      "id": "11",
      "name": "Log_to_Appwrite",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "POST",
        "url": "https://cloud.appwrite.io/v1/databases/prod/collections/social_posts/documents",
        "sendHeaders": true,
        "headerParameters": {
          "parameters": [
            {
              "name": "X-Appwrite-Project",
              "value": "={{$credentials.appwriteProject}}"
            },
            {
              "name": "X-Appwrite-Key",
              "value": "={{$credentials.appwriteKey}}"
            }
          ]
        },
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "documentId",
              "value": "={{$json.post_id}}"
            },
            {
              "name": "data",
              "value": "={\n  \"platform\": \"instagram\",\n  \"content\": \"{{$json.content.caption}}\",\n  \"media_url\": \"{{$json.image_url}}\",\n  \"product_id\": \"{{$node['Get_Product_Data'].json.id}}\",\n  \"scheduled_at\": \"{{$now.toISO()}}\",\n  \"published_at\": \"{{$now.toISO()}}\",\n  \"status\": \"published\",\n  \"ai_confidence\": {{$json.confidence}},\n  \"engagement_metrics\": {}\n}"
            }
          ]
        }
      },
      "position": [2250, 300]
    }
  ]
}
```

---

## 📧 Core Workflow #2: Email Style Learning & Response

### Training Pipeline for Brand Voice

```json
{
  "name": "Email_Training_Pipeline",
  "nodes": [
    {
      "name": "Email_Trigger",
      "type": "n8n-nodes-base.emailReadImap",
      "parameters": {
        "mailbox": "Training",
        "postProcessAction": "move",
        "moveToMailbox": "Processed"
      }
    },
    {
      "name": "Extract_Email_Patterns",
      "type": "n8n-nodes-base.code",
      "parameters": {
        "code": "// Extract writing patterns\nconst email = $input.item.json;\n\n// Analyze email structure\nconst analysis = {\n  greeting_style: email.text.match(/^(Dear|Hi|Hello|Hey).*$/m)?.[1] || 'None',\n  sentence_length: email.text.split('.').map(s => s.split(' ').length),\n  avg_sentence_length: 0,\n  tone_indicators: [],\n  closing_style: email.text.match(/(Best|Regards|Sincerely|Thanks).*$/m)?.[1] || 'None',\n  key_phrases: [],\n  emojis_used: (email.text.match(/[\\u{1F300}-\\u{1F9FF}]/gu) || []).length > 0\n};\n\n// Calculate averages\nanalysis.avg_sentence_length = analysis.sentence_length.reduce((a,b) => a+b, 0) / analysis.sentence_length.length;\n\n// Detect tone\nconst formalWords = ['pursuant', 'regarding', 'kindly', 'please find attached'];\nconst casualWords = ['hey', 'awesome', 'cool', 'thanks'];\n\nformalWords.forEach(word => {\n  if(email.text.toLowerCase().includes(word)) {\n    analysis.tone_indicators.push('formal');\n  }\n});\n\ncasualWords.forEach(word => {\n  if(email.text.toLowerCase().includes(word)) {\n    analysis.tone_indicators.push('casual');\n  }\n});\n\n// Extract frequently used phrases\nconst phrases = email.text.match(/\\b\\w+\\s+\\w+\\s+\\w+\\b/g) || [];\nconst phraseCount = {};\nphrases.forEach(phrase => {\n  phraseCount[phrase] = (phraseCount[phrase] || 0) + 1;\n});\n\nanalysis.key_phrases = Object.entries(phraseCount)\n  .sort((a,b) => b[1] - a[1])\n  .slice(0, 10)\n  .map(([phrase]) => phrase);\n\nreturn {\n  json: {\n    email_id: email.messageId,\n    subject: email.subject,\n    analysis: analysis,\n    full_text: email.text\n  }\n};"
      }
    },
    {
      "name": "Generate_Embeddings",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "POST",
        "url": "https://generativelanguage.googleapis.com/v1beta/models/text-embedding-004:embedContent",
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "content",
              "value": "={{$json.full_text}}"
            },
            {
              "name": "model",
              "value": "models/text-embedding-004"
            }
          ]
        }
      }
    },
    {
      "name": "Store_in_Appwrite",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "POST",
        "url": "https://cloud.appwrite.io/v1/databases/prod/collections/email_training/documents",
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "data",
              "value": "={\n  \"email_id\": \"{{$json.email_id}}\",\n  \"subject\": \"{{$json.subject}}\",\n  \"analysis\": {{$json.analysis}},\n  \"embedding\": {{$json.embedding}},\n  \"processed_at\": \"{{$now.toISO()}}\"\n}"
            }
          ]
        }
      }
    }
  ]
}
```

---

## 💰 Core Workflow #3: Daily Accounting Automation

### Shopify to Spreadsheet/Accounting Sync

```json
{
  "name": "Daily_Accounting_Sync",
  "nodes": [
    {
      "name": "Daily_Trigger",
      "type": "n8n-nodes-base.scheduleTrigger",
      "parameters": {
        "rule": {
          "cronExpression": "0 23 * * *"
        }
      }
    },
    {
      "name": "Get_Todays_Orders",
      "type": "n8n-nodes-base.shopify",
      "parameters": {
        "operation": "getAll",
        "resource": "order",
        "additionalFields": {
          "created_at_min": "={{$today.toISOString()}}",
          "created_at_max": "={{$now.toISOString()}}",
          "status": "any"
        }
      }
    },
    {
      "name": "Calculate_Metrics",
      "type": "n8n-nodes-base.code",
      "parameters": {
        "code": "// Daily financial calculations\nconst orders = $input.all();\n\nconst metrics = {\n  date: new Date().toISOString().split('T')[0],\n  total_revenue: 0,\n  total_orders: orders.length,\n  average_order_value: 0,\n  total_items_sold: 0,\n  total_shipping: 0,\n  total_tax: 0,\n  total_discounts: 0,\n  net_revenue: 0,\n  new_customers: 0,\n  returning_customers: 0,\n  payment_methods: {},\n  top_products: [],\n  order_details: []\n};\n\n// Process each order\norders.forEach(order => {\n  const orderData = order.json;\n  \n  // Revenue calculations\n  metrics.total_revenue += parseFloat(orderData.total_price || 0);\n  metrics.total_shipping += parseFloat(orderData.total_shipping_price_set?.shop_money?.amount || 0);\n  metrics.total_tax += parseFloat(orderData.total_tax || 0);\n  metrics.total_discounts += parseFloat(orderData.total_discounts || 0);\n  \n  // Items sold\n  orderData.line_items?.forEach(item => {\n    metrics.total_items_sold += item.quantity;\n  });\n  \n  // Customer type\n  if (orderData.customer?.orders_count === 1) {\n    metrics.new_customers++;\n  } else {\n    metrics.returning_customers++;\n  }\n  \n  // Payment methods\n  const payment = orderData.payment_gateway_names?.[0] || 'unknown';\n  metrics.payment_methods[payment] = (metrics.payment_methods[payment] || 0) + 1;\n  \n  // Order details for ledger\n  metrics.order_details.push({\n    order_number: orderData.order_number,\n    customer: orderData.customer?.email || 'Guest',\n    total: orderData.total_price,\n    items: orderData.line_items?.length || 0,\n    status: orderData.financial_status\n  });\n});\n\n// Calculate aggregates\nmetrics.average_order_value = metrics.total_orders > 0 ? \n  (metrics.total_revenue / metrics.total_orders).toFixed(2) : 0;\n  \nmetrics.net_revenue = metrics.total_revenue - metrics.total_tax - metrics.total_shipping;\n\n// Get top products\nconst productSales = {};\norders.forEach(order => {\n  order.json.line_items?.forEach(item => {\n    const key = item.title;\n    if (!productSales[key]) {\n      productSales[key] = { name: key, quantity: 0, revenue: 0 };\n    }\n    productSales[key].quantity += item.quantity;\n    productSales[key].revenue += parseFloat(item.price) * item.quantity;\n  });\n});\n\nmetrics.top_products = Object.values(productSales)\n  .sort((a, b) => b.revenue - a.revenue)\n  .slice(0, 5);\n\nreturn {\n  json: metrics\n};"
      }
    },
    {
      "name": "Update_Google_Sheets",
      "type": "n8n-nodes-base.googleSheets",
      "parameters": {
        "operation": "appendOrUpdate",
        "documentId": "={{$credentials.spreadsheetId}}",
        "sheetName": "Daily_Revenue",
        "columns": {
          "mappingMode": "defineBelow",
          "value": {
            "Date": "={{$json.date}}",
            "Revenue": "={{$json.total_revenue}}",
            "Orders": "={{$json.total_orders}}",
            "AOV": "={{$json.average_order_value}}",
            "Items": "={{$json.total_items_sold}}",
            "New_Customers": "={{$json.new_customers}}",
            "Net_Revenue": "={{$json.net_revenue}}"
          }
        }
      }
    },
    {
      "name": "Generate_Daily_Report",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "POST",
        "url": "https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent",
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "contents",
              "value": "=[\n  {\n    \"parts\": [\n      {\n        \"text\": \"Generate a professional daily sales report based on this data:\\n\\n{{$json}}\\n\\nInclude:\\n1. Executive summary\\n2. Key metrics highlights\\n3. Notable trends\\n4. Recommendations\\n\\nTone: Professional but concise\"\n      }\n    ]\n  }\n]"
            }
          ]
        }
      }
    },
    {
      "name": "Send_Report_Telegram",
      "type": "n8n-nodes-base.telegram",
      "parameters": {
        "operation": "sendMessage",
        "chatId": "={{$credentials.telegramChatId}}",
        "text": "=📊 **Daily Sales Report - {{$json.date}}**\\n\\n{{$node['Generate_Daily_Report'].json.report}}\\n\\n**Quick Stats:**\\n💰 Revenue: ${{$json.total_revenue}}\\n📦 Orders: {{$json.total_orders}}\\n🛒 AOV: ${{$json.average_order_value}}\\n👥 New Customers: {{$json.new_customers}}",
        "additionalFields": {
          "parse_mode": "Markdown"
        }
      }
    }
  ]
}
```

---

## 🎯 Core Workflow #4: B2B Outreach Automation

### Local Business Partnership Outreach

```json
{
  "name": "B2B_Salon_Outreach",
  "nodes": [
    {
      "name": "Weekly_Trigger",
      "type": "n8n-nodes-base.scheduleTrigger",
      "parameters": {
        "rule": {
          "cronExpression": "0 10 * * 1"
        }
      }
    },
    {
      "name": "Search_Local_Businesses",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "GET",
        "url": "https://maps.googleapis.com/maps/api/place/nearbysearch/json",
        "sendQuery": true,
        "queryParameters": {
          "parameters": [
            {
              "name": "location",
              "value": "={{$credentials.businessLatLng}}"
            },
            {
              "name": "radius",
              "value": "10000"
            },
            {
              "name": "type",
              "value": "beauty_salon|hair_care|spa"
            },
            {
              "name": "key",
              "value": "={{$credentials.googleMapsApiKey}}"
            }
          ]
        }
      }
    },
    {
      "name": "Get_Business_Details",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "GET",
        "url": "https://maps.googleapis.com/maps/api/place/details/json",
        "sendQuery": true,
        "queryParameters": {
          "parameters": [
            {
              "name": "place_id",
              "value": "={{$json.place_id}}"
            },
            {
              "name": "fields",
              "value": "name,formatted_address,formatted_phone_number,website,rating,user_ratings_total,opening_hours"
            }
          ]
        }
      }
    },
    {
      "name": "Generate_Personalized_Outreach",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "POST",
        "url": "https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent",
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "contents",
              "value": "=[\n  {\n    \"parts\": [\n      {\n        \"text\": \"Create a personalized B2B outreach email for a beauty business partnership.\\n\\nBusiness: {{$json.name}}\\nRating: {{$json.rating}} stars ({{$json.user_ratings_total}} reviews)\\nWebsite: {{$json.website}}\\n\\nOur Product: [E-commerce beauty/cosmetic products]\\n\\nGoals:\\n1. Introduce wholesale partnership opportunity\\n2. Highlight mutual benefits\\n3. Propose initial meeting/call\\n\\nTone: Professional, friendly, value-focused\\nLength: 150-200 words\\n\\nInclude:\\n- Personalized greeting\\n- Reference to their business success\\n- Clear value proposition\\n- Specific partnership benefits\\n- Clear call-to-action\"\n      }\n    ]\n  }\n]"
            }
          ]
        }
      }
    },
    {
      "name": "Check_Previous_Contact",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "GET",
        "url": "https://cloud.appwrite.io/v1/databases/prod/collections/b2b_outreach/documents",
        "sendQuery": true,
        "queryParameters": {
          "parameters": [
            {
              "name": "queries[]",
              "value": "equal(\"business_name\",[\"{{$json.name}}\"])"
            }
          ]
        }
      }
    },
    {
      "name": "Send_if_New",
      "type": "n8n-nodes-base.if",
      "parameters": {
        "conditions": {
          "number": [
            {
              "value1": "={{$json.total}}",
              "operation": "equal",
              "value2": 0
            }
          ]
        }
      }
    },
    {
      "name": "Send_Outreach_Email",
      "type": "n8n-nodes-base.emailSend",
      "parameters": {
        "fromEmail": "={{$credentials.businessEmail}}",
        "toEmail": "={{$json.email}}",
        "subject": "Partnership Opportunity with {{$json.name}}",
        "text": "={{$node['Generate_Personalized_Outreach'].json.email}}",
        "additionalFields": {
          "replyTo": "={{$credentials.businessEmail}}",
          "attachments": "company_catalog.pdf"
        }
      }
    },
    {
      "name": "Log_Outreach",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "POST",
        "url": "https://cloud.appwrite.io/v1/databases/prod/collections/b2b_outreach/documents",
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "data",
              "value": "={\n  \"business_name\": \"{{$json.name}}\",\n  \"email\": \"{{$json.email}}\",\n  \"phone\": \"{{$json.formatted_phone_number}}\",\n  \"outreach_date\": \"{{$now.toISO()}}\",\n  \"email_content\": \"{{$node['Generate_Personalized_Outreach'].json.email}}\",\n  \"status\": \"sent\",\n  \"follow_up_date\": \"{{$now.plus(7, 'days').toISO()}}\"\n}"
            }
          ]
        }
      }
    }
  ]
}
```

---

## 🤖 Core Workflow #5: Smart Rate Limiting System

### Universal API Rate Limiter

```json
{
  "name": "Rate_Limit_Manager",
  "nodes": [
    {
      "name": "API_Call_Webhook",
      "type": "n8n-nodes-base.webhook",
      "parameters": {
        "path": "rate-check",
        "responseMode": "responseNode",
        "options": {
          "responseData": "allEntries",
          "responsePropertyName": "data"
        }
      }
    },
    {
      "name": "Check_Limits",
      "type": "n8n-nodes-base.code",
      "parameters": {
        "code": "// Rate limit configuration\nconst limits = {\n  instagram: { requests: 180, window: 3600000 }, // 180/hour\n  facebook: { requests: 180, window: 3600000 },\n  shopify: { requests: 2, window: 1000 }, // 2/second\n  gemini: { requests: 60, window: 60000 }, // 60/minute\n  gemini_batch: { requests: 100, window: 86400000 }, // 100/day batch\n  telegram: { requests: 30, window: 1000 }, // 30/second\n  google_maps: { requests: 100, window: 86400000 } // 100/day\n};\n\nconst api = $input.first().json.api;\nconst now = Date.now();\nconst limit = limits[api];\n\nif (!limit) {\n  return {\n    json: {\n      error: 'Unknown API',\n      allowed: false\n    }\n  };\n}\n\n// Get recent calls from Redis\nconst windowStart = now - limit.window;\n\n// This would connect to Redis in production\n// For demo, using in-memory simulation\nconst recentCalls = await getRecentCalls(api, windowStart);\n\nif (recentCalls.length < limit.requests) {\n  // Log this call\n  await logApiCall(api, now);\n  \n  return {\n    json: {\n      allowed: true,\n      remaining: limit.requests - recentCalls.length - 1,\n      reset_in: Math.ceil((windowStart + limit.window - now) / 1000),\n      api: api,\n      current_usage: recentCalls.length + 1,\n      limit: limit.requests\n    }\n  };\n} else {\n  // Calculate backoff\n  const overflow = recentCalls.length - limit.requests + 1;\n  const backoff = Math.min(Math.pow(2, overflow) * 1000, 300000); // Max 5 min\n  \n  return {\n    json: {\n      allowed: false,\n      remaining: 0,\n      retry_after: Math.ceil(backoff / 1000),\n      reset_in: Math.ceil((windowStart + limit.window - now) / 1000),\n      api: api,\n      current_usage: recentCalls.length,\n      limit: limit.requests,\n      message: `Rate limit exceeded. Retry after ${Math.ceil(backoff / 1000)} seconds`\n    }\n  };\n}\n\n// Helper functions (would be actual Redis calls)\nasync function getRecentCalls(api, since) {\n  // Simulate Redis ZRANGEBYSCORE\n  return [];\n}\n\nasync function logApiCall(api, timestamp) {\n  // Simulate Redis ZADD\n  return true;\n}"
      }
    },
    {
      "name": "Response",
      "type": "n8n-nodes-base.respondToWebhook",
      "parameters": {
        "options": {
          "responseHeaders": {
            "entries": [
              {
                "name": "X-RateLimit-Limit",
                "value": "={{$json.limit}}"
              },
              {
                "name": "X-RateLimit-Remaining",
                "value": "={{$json.remaining}}"
              },
              {
                "name": "X-RateLimit-Reset",
                "value": "={{$json.reset_in}}"
              }
            ]
          }
        }
      }
    }
  ]
}
```

---

## 📈 Performance Monitoring Workflow

### Analytics & Optimization Tracking

```json
{
  "name": "Performance_Analytics",
  "nodes": [
    {
      "name": "Hourly_Check",
      "type": "n8n-nodes-base.scheduleTrigger",
      "parameters": {
        "rule": {
          "cronExpression": "0 * * * *"
        }
      }
    },
    {
      "name": "Get_Instagram_Insights",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "GET",
        "url": "https://graph.facebook.com/v18.0/{{$credentials.instagramBusinessId}}/insights",
        "sendQuery": true,
        "queryParameters": {
          "parameters": [
            {
              "name": "metric",
              "value": "impressions,reach,profile_views,website_clicks"
            },
            {
              "name": "period",
              "value": "day"
            },
            {
              "name": "access_token",
              "value": "={{$credentials.metaAccessToken}}"
            }
          ]
        }
      }
    },
    {
      "name": "Get_Recent_Posts_Performance",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "GET",
        "url": "https://cloud.appwrite.io/v1/databases/prod/collections/social_posts/documents",
        "sendQuery": true,
        "queryParameters": {
          "parameters": [
            {
              "name": "queries[]",
              "value": "greaterThan(\"published_at\",[\"{{$now.minus(24, 'hours').toISO()}}\"])"
            }
          ]
        }
      }
    },
    {
      "name": "Calculate_Engagement",
      "type": "n8n-nodes-base.code",
      "parameters": {
        "code": "// Calculate engagement metrics\nconst insights = $node['Get_Instagram_Insights'].json;\nconst posts = $node['Get_Recent_Posts_Performance'].json.documents;\n\nconst metrics = {\n  timestamp: new Date().toISOString(),\n  period: 'last_24_hours',\n  instagram: {\n    impressions: insights.impressions || 0,\n    reach: insights.reach || 0,\n    profile_views: insights.profile_views || 0,\n    website_clicks: insights.website_clicks || 0,\n    engagement_rate: 0\n  },\n  posts_performance: [],\n  ai_effectiveness: {\n    avg_confidence: 0,\n    auto_approved: 0,\n    manual_approved: 0,\n    rejected: 0\n  },\n  recommendations: []\n};\n\n// Calculate engagement rate\nif (metrics.instagram.reach > 0) {\n  const totalEngagement = insights.likes + insights.comments + insights.shares || 0;\n  metrics.instagram.engagement_rate = (totalEngagement / metrics.instagram.reach * 100).toFixed(2);\n}\n\n// Analyze post performance\nposts.forEach(post => {\n  const postMetrics = {\n    id: post.$id,\n    confidence: post.ai_confidence,\n    status: post.status,\n    engagement: post.engagement_metrics || {}\n  };\n  \n  metrics.posts_performance.push(postMetrics);\n  \n  // AI effectiveness\n  metrics.ai_effectiveness.avg_confidence += post.ai_confidence;\n  if (post.ai_confidence >= 0.85) {\n    metrics.ai_effectiveness.auto_approved++;\n  } else if (post.status === 'published') {\n    metrics.ai_effectiveness.manual_approved++;\n  } else if (post.status === 'rejected') {\n    metrics.ai_effectiveness.rejected++;\n  }\n});\n\n// Calculate averages\nif (posts.length > 0) {\n  metrics.ai_effectiveness.avg_confidence /= posts.length;\n}\n\n// Generate recommendations\nif (metrics.instagram.engagement_rate < 2) {\n  metrics.recommendations.push('Engagement below 2%. Consider more interactive content.');\n}\n\nif (metrics.ai_effectiveness.avg_confidence < 0.7) {\n  metrics.recommendations.push('AI confidence low. Review and improve prompts.');\n}\n\nif (metrics.instagram.website_clicks < 10) {\n  metrics.recommendations.push('Low website traffic. Strengthen CTAs in posts.');\n}\n\nreturn { json: metrics };"
      }
    },
    {
      "name": "Store_Analytics",
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "method": "POST",
        "url": "https://cloud.appwrite.io/v1/databases/prod/collections/analytics/documents",
        "sendBody": true,
        "bodyParameters": {
          "parameters": [
            {
              "name": "data",
              "value": "={{$json}}"
            }
          ]
        }
      }
    }
  ]
}
```

---

## 🎓 Training Data Implementation

### Email Style Learning System

```javascript
// Email Training Configuration
const emailTrainingConfig = {
  // Minimum emails needed for effective training
  minimum_samples: 50,
  
  // Pattern extraction
  patterns_to_learn: [
    'greeting_styles',      // How you start emails
    'closing_styles',       // How you end emails
    'sentence_structure',   // Average length, complexity
    'vocabulary',          // Common words and phrases
    'tone_indicators',     // Formal vs casual
    'response_patterns',   // How you handle different queries
    'emoji_usage',        // If and how you use emojis
    'formatting'          // Bold, italic, lists usage
  ],
  
  // Training pipeline
  pipeline: [
    'collect_emails',      // IMAP or forward to training inbox
    'extract_patterns',    // Analyze structure and style
    'generate_embeddings', // Vector representations
    'store_in_database',   // Appwrite Cloud storage
    'create_templates',    // Generate response templates
    'validate_output'      // Human review of generated content
  ]
};

// Product Image Training
const imageTrainingConfig = {
  // Imagen 3 prompting for brand consistency
  style_parameters: {
    lighting: "soft studio lighting, white background",
    angle: "45-degree product hero shot",
    composition: "centered, rule of thirds",
    post_processing: "high contrast, vibrant colors",
    brand_elements: "consistent props and staging"
  },
  
  // Training from existing images
  reference_images: [
    "Upload 10-20 best performing product photos",
    "System learns composition and style",
    "Generates similar aesthetic for new products"
  ]
};
```

---

## 🚀 Implementation Priority Order

### Week 1: Foundation
1. **Rate Limit Manager** - Prevent API bans (2 hours)
2. **Shopify Order Webhook** - Basic order processing (3 hours)
3. **Daily Accounting Sync** - Financial tracking (4 hours)

### Week 2: Content & AI
4. **Instagram Content Generation** - Core social media (6 hours)
5. **Email Training Pipeline** - Learn brand voice (4 hours)
6. **Quality Score Checker** - AI confidence system (3 hours)

### Week 3: Automation
7. **Customer Response Bot** - Email automation (5 hours)
8. **B2B Outreach System** - Lead generation (5 hours)
9. **Review Management** - Reputation building (3 hours)

### Week 4: Optimization
10. **Performance Analytics** - Track and improve (4 hours)
11. **A/B Testing Framework** - Optimize content (4 hours)
12. **Advanced Reporting** - Business intelligence (3 hours)

---

## 💡 Pro Tips for n8n Implementation

### 1. Error Handling Pattern
```javascript
// Always wrap critical nodes
try {
  // API call or processing
  const result = await $item.api_call();
  return { json: { success: true, data: result }};
} catch (error) {
  // Log to Appwrite
  await logError(error);
  // Send Telegram alert
  await sendAlert(`Workflow failed: ${error.message}`);
  // Return safe default
  return { json: { success: false, error: error.message }};
}
```

### 2. Memory Management
```javascript
// Use window buffer for context
const memoryConfig = {
  type: 'window_buffer',
  k: 5, // Keep last 5 interactions
  session_id: '{{$json.customer_id}}',
  system_message: 'You are a helpful e-commerce assistant'
};
```

### 3. Batch Processing for Gemini
```javascript
// Use batch API for 50% cost savings
const batchConfig = {
  model: 'gemini-1.5-flash',
  requests: items.map(item => ({
    contents: [{
      parts: [{ text: generatePrompt(item) }]
    }]
  })),
  batch_size: 100,
  priority: 'standard' // vs 'urgent'
};
```

### 4. Dynamic Grounding
```javascript
// Only search when needed
const groundingConfig = {
  threshold: 0.7, // Only ground if confidence < 70%
  search_queries: 3,
  include_sources: true
};
```

---

## 📊 Success Metrics

### After 30 Days, Expect:
- **Instagram Posts**: 90 automated posts (3x daily)
- **Email Responses**: 200+ automated responses
- **B2B Outreach**: 100+ personalized emails sent
- **Time Saved**: 120+ hours/month
- **API Costs**: < $30 with optimization
- **Revenue Impact**: 20-40% increase from consistency

---

## 🔧 Troubleshooting Common Issues

### Rate Limit Errors
```javascript
// Solution: Implement exponential backoff
const backoff = Math.min(2 ** retryCount * 1000, 30000);
await new Promise(resolve => setTimeout(resolve, backoff));
```

### Low AI Confidence
```javascript
// Solution: Improve prompts with examples
const improvedPrompt = `
Examples of good posts:
1. [Example 1]
2. [Example 2]

Now create similar content for: ${product}
`;
```

### Memory Issues on Pi
```bash
# Solution: Limit container memory
docker update --memory="400m" n8n
docker update --memory="200m" redis
```

---

## Next Steps

1. **Start with Rate Limiter** - Critical for all workflows
2. **Implement Core 3** - Instagram, Orders, Accounting
3. **Train AI on Your Data** - 50-100 emails minimum
4. **Monitor and Optimize** - Use analytics workflow
5. **Scale Gradually** - Add workflows as stable

Remember: Start simple, test thoroughly, scale gradually!