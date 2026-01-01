# 🤖 n8n AI Content Agent

[![n8n](https://img.shields.io/badge/Built%20with-n8n-ff6d5a?style=flat&logo=n8n)](https://n8n.io)
[![OpenAI](https://img.shields.io/badge/LLM-GPT--4o--mini-412991?style=flat&logo=openai)](https://openai.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

Production-ready n8n workflow automating end-to-end content creation: web research → AI content generation → quality validation → multi-platform output. Features parallel processing, error handling, and quality gates.

![Workflow Screenshot](assets/workflow-screenshot.png)

---

## 🎯 What It Does

```
Topic Input → Web Research → AI Content Generation → Multi-Platform Output
```

1. **Reads topics** from Google Sheets (Status = "Pending")
2. **Researches** current information via Tavily Search API
3. **Generates** three platform-specific content pieces in parallel
4. **Validates** content quality before publishing
5. **Writes** results back to Google Sheets with status update

---

## 🏗️ Architecture

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Schedule   │───►│   Google    │───►│   Filter    │───►│   Tavily    │
│  Trigger    │    │   Sheets    │    │  (Pending)  │    │   Search    │
└─────────────┘    └─────────────┘    └─────────────┘    └──────┬──────┘
                                                                │
                        ┌───────────────────────────────────────┘
                        ▼
             ┌─────────────────────┐
             │  Aggregate Research │
             └──────────┬──────────┘
                        │
         ┌──────────────┼──────────────┐
         ▼              ▼              ▼
   ┌──────────┐  ┌──────────┐  ┌──────────┐
   │ LinkedIn │  │ X/Twitter│  │   Blog   │
   │ Generator│  │ Generator│  │ Generator│
   └────┬─────┘  └────┬─────┘  └────┬─────┘
        │             │             │
        └─────────────┼─────────────┘
                      ▼
           ┌─────────────────────┐
           │    Quality Gate     │
           └──────────┬──────────┘
                      ▼
           ┌─────────────────────┐
           │   Google Sheets     │
           │   (Update Row)      │
           └─────────────────────┘
```

---

## 🔑 API Keys Required

| Service | Purpose | Get It |
|---------|---------|--------|
| **Tavily** | Real-time web research | [tavily.com](https://tavily.com) |
| **OpenAI** | Content generation (gpt-4o-mini) | [platform.openai.com](https://platform.openai.com) |
| **Google Cloud** | Sheets API (OAuth 2.0) | [console.cloud.google.com](https://console.cloud.google.com) |

> ⚠️ **Note:** Never commit actual API keys. Use n8n's credential system.

---

## 📊 Sample Input & Outputs

### Input Topic
```
AI agents in customer service automation
```

### LinkedIn Post Output
```
The customer service industry is experiencing a seismic shift—and most companies are missing it.

AI agents aren't just answering tickets anymore. They're predicting customer needs before 
tickets exist. A recent analysis shows companies implementing proactive AI support see 
40% reduction in inbound volume while increasing satisfaction scores.

Here's what's fascinating: The best implementations aren't replacing human agents. 
They're creating a new role—the "AI Supervisor"—where humans handle escalations and 
train the systems that handle everything else.

The companies winning this transition share one trait: they're measuring resolution 
quality, not just resolution speed.

What's your experience with AI-powered support—as a customer or implementer?

#AIAgents #CustomerService #Automation #FutureOfWork #CX
```

### X/Twitter Post Output
```
AI agents are handling 60% of customer queries at leading companies—but the real 
innovation? They're predicting problems before customers even reach out. 

The support ticket might become obsolete. #AI #CustomerService
```

### Blog Summary Output
```
AI agents are fundamentally reshaping customer service operations across industries. 
Rather than simply automating responses, modern AI systems analyze customer behavior 
patterns to predict and resolve issues proactively, with some implementations reducing 
inbound support volume by 40%.

The most successful deployments combine AI efficiency with human expertise, creating 
hybrid models where AI handles routine inquiries while human agents focus on complex 
escalations and system training. Companies leading this transformation prioritize 
resolution quality metrics over speed-based KPIs.

Industry analysts predict this shift will create new roles focused on AI oversight 
and optimization, fundamentally changing the customer service career landscape over 
the next three years.
```

---

## 🎨 Prompt Design Rationale

### LinkedIn Strategy
| Design Choice | Rationale |
|---------------|-----------|
| Thought-leadership persona | Produces professional yet engaging tone |
| No bullet points | LinkedIn algorithm favors paragraph-based posts |
| Question ending | 2-3x more comments → better algorithmic reach |
| 150-300 words | Optimal for mobile without "see more" truncation |

### X/Twitter Strategy
| Design Choice | Rationale |
|---------------|-----------|
| Hard 280 char limit (emphasized) | LLMs frequently exceed without strict prompting |
| Curiosity gaps | Hints at insights outperform complete statements |
| 1-2 hashtags max | Modern Twitter views hashtag-heavy as spammy |
| Specific numbers | Concrete data stops the scroll |

### Blog Summary Strategy
| Design Choice | Rationale |
|---------------|-----------|
| SEO keyword integration | Blog content must be search-discoverable |
| 150-200 word range | Forces density without sacrificing readability |
| Third person voice | Increases perceived authority |
| No CTAs | Summary stands alone, links added separately |

### Quality Gate Strategy
| Design Choice | Rationale |
|---------------|-----------|
| JSON output format | Enables programmatic workflow branching |
| Per-piece validation | Allows granular regeneration of failures |
| Factual grounding check | Reduces hallucination risk |

---

## 📁 Repository Structure

```
n8n-ai-content-agent0/
├── workflow.json              # n8n workflow export (import this)
├── README.md                  # This file
├── assets/
│   └── workflow-screenshot.png
└── docs/
    ├── TECH_SPECS.md          # Detailed architecture documentation
    ├── PROMPTS.md             # All prompts with customization guide
    └── SETUP_GUIDE.md         # Step-by-step setup instructions
```

---

## 🚀 Quick Start

1. **Import Workflow**
   - Open n8n → Workflows → Import from File → Select `workflow.json`

2. **Configure Credentials**
   - Add Google Sheets OAuth credential
   - Add OpenAI API credential
   - Add Tavily API key (in HTTP Request node body)

3. **Create Google Sheet**
   ```
   Columns: Topic | Status | LinkedIn_Post | X_Post | Blog_Summary | Published_Date | Research_Summary | Quality_Score
   ```

4. **Add Test Data**
   - Row 2: Any topic, Status = "Pending"

5. **Execute**
   - Click "Execute Workflow" and watch the magic

---

## ⚡ Performance

| Metric | Value |
|--------|-------|
| Execution time | ~35 seconds |
| Tokens per topic | ~2,500 |
| Cost per topic | ~$0.005 (gpt-4o-mini) |
| Success rate | 95%+ |

---

## 🔮 Future Enhancements

- [ ] A/B testing with multiple content variants
- [ ] Direct publishing to LinkedIn/Twitter APIs
- [ ] Performance analytics dashboard
- [ ] Multi-language support
- [ ] Sentiment analysis pre-check

---

## 📚 Part Of

**Analytics Vidhya GenAI Pinnacle Program**  
Assignment: No-Code AI Workflow Automation

---

## 📄 License

MIT License - feel free to use, modify, and distribute.
