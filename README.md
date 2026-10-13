# AI-Powered Lead Qualification Automation for a Clothing Brand

## Overview

This project is an AI-powered lead qualification workflow built using **n8n** and **Google Gemini**. It automates the process of responding to customer inquiries and classifying potential customers according to their likelihood of making a purchase.

The automation helps a clothing brand handle incoming inquiries more efficiently by generating personalized customer responses, assigning lead scores, and categorizing leads as either HOT or COLD.

By automating these repetitive tasks, the workflow can help improve response times, streamline sales follow-ups, and allow the sales team to prioritize customers with stronger purchasing intent.

## Problem Statement

Clothing brands receive inquiries from customers interested in products, prices, sizes, colours, availability, delivery, and ordering procedures.

Manually reviewing each inquiry, composing a response, and determining its sales potential can be time-consuming. Without a structured qualification process, businesses may miss promising leads or spend too much time following up with customers who are only browsing.

This automation addresses the problem by using AI to generate customer responses and assess purchasing intent based on predefined qualification criteria.

## Objectives

- Automate responses to incoming customer inquiries.
- Use AI to identify customer purchasing intent.
- Classify leads as HOT or COLD.
- Assign a numerical lead score between 0 and 100.
- Generate a reason for each lead classification.
- Recommend an appropriate next action for the sales team.
- Produce structured output that can be used by downstream n8n nodes.

## Tools and Technologies

- **n8n:** Workflow automation and orchestration.
- **Google Gemini:** AI-powered inquiry analysis, response generation, and lead qualification.
- **JavaScript:** Processing and parsing the AI-generated JSON in the n8n Code node.
- **JSON:** Standardized data format for passing results between workflow nodes.
- **n8n IF node:** Conditional routing based on lead classification.

## How the Automation Works

The workflow follows these steps:

1. **Trigger:** A customer inquiry enters the workflow through the configured input source.
2. **AI analysis:** Google Gemini analyzes the customer's message and generates a personalized response.
3. **Lead qualification:** The AI determines the customer's purchasing intent and assigns a lead score.
4. **JSON parsing:** The Code node extracts the AI-generated JSON string and converts it into separate, accessible fields.
5. **Lead routing:** An IF node checks whether the lead is HOT or COLD.
6. **Next action:** The workflow can direct hot leads to priority sales follow-up and cold leads to a customer-nurturing process.

## Lead Qualification Criteria

The workflow uses purchasing signals to determine lead quality.

| Classification | Criteria | Recommended action |
|---|---|---|
| HOT | Strong purchasing intent, a clear intention to order, or specific purchase requirements | Prioritize follow-up and assist with the purchase |
| COLD | General inquiries, casual browsing, or insufficient evidence of buying intent | Provide information and nurture the relationship |

The AI also assigns a lead score from 0 to 100.

- **70–100:** HOT lead
- **0–69:** COLD lead

These thresholds can be adjusted as the business gathers data on actual customer conversions.

## AI Output Structure

Google Gemini returns a JSON object containing the following fields:

```json
{
  "customer_response": "A personalized response to the customer",
  "lead_status": "HOT",
  "lead_score": 85,
  "qualification_reason": "The customer has expressed a clear intention to purchase.",
  "purchase_intent": "High",
  "next_action": "Prioritize follow-up and assist the customer with ordering."
}
```

### Field descriptions

| Field | Description |
|---|---|
| `customer_response` | Message intended for the customer |
| `lead_status` | HOT or COLD classification |
| `lead_score` | Numerical score indicating lead quality |
| `qualification_reason` | Explanation supporting the classification |
| `purchase_intent` | High, Medium, or Low |
| `next_action` | Recommended action for the sales team |

## JavaScript Code Node

The Code node extracts the response from the Gemini output, parses the JSON string, validates the required fields, normalizes the lead status, and returns the information as separate JSON properties.

This makes the output easier to map into subsequent n8n nodes.

Example:

```javascript
const output = $input.first().json;

let text = output.content?.parts?.[0]?.text;

if (!text) {
  throw new Error("AI response text was not found.");
}

text = text
  .replace(/^```(?:json)?\s*/i, "")
  .replace(/\s*```$/, "")
  .trim();

let parsed;

try {
  parsed = JSON.parse(text);
} catch (error) {
  throw new Error("Could not parse the AI response as JSON.");
}

const requiredFields = [
  "customer_response",
  "lead_status",
  "lead_score",
  "qualification_reason",
  "purchase_intent",
  "next_action"
];

for (const field of requiredFields) {
  if (parsed[field] === undefined || parsed[field] === null) {
    throw new Error(`Missing required field: ${field}`);
  }
}

const leadStatus = String(parsed.lead_status).toUpperCase();

if (!["HOT", "COLD"].includes(leadStatus)) {
  throw new Error(`Invalid lead status: ${parsed.lead_status}`);
}

return [
  {
    json: {
      customer_response: parsed.customer_response,
      lead_status: leadStatus,
      lead_score: Number(parsed.lead_score),
      qualification_reason: parsed.qualification_reason,
      purchase_intent: parsed.purchase_intent,
      next_action: parsed.next_action
    }
  }
];
```

**Note:** This implementation validates the required fields and lead-status values. Additional validation can be added to enforce score ranges, allowed purchase-intent values, and consistency between the score and classification.

## Example Workflow Routing

The workflow can be extended using an IF node with the following condition:

```text
{{ $json.lead_status }} is equal to HOT
```

- **True branch:** Send a notification to the sales team, record the lead, and prioritize follow-up.
- **False branch:** Send the generated response and optionally add the customer to a nurturing list.

## Business Benefits

- **Time efficiency:** Reduces manual response writing and initial lead assessment.
- **Faster engagement:** Enables quicker responses to customer inquiries.
- **Sales prioritization:** Helps identify customers showing stronger purchase intent.
- **Consistent qualification:** Applies a common set of lead-scoring rules.
- **Scalability:** Provides a foundation for processing larger volumes of inquiries.
- **Structured data:** Makes lead information easier to store, track, and analyze.

## Limitations and Considerations

- AI-generated classifications may occasionally be inaccurate.
- Lead scores are estimates of purchasing intent, not guarantees of conversion.
- The model should not invent prices, stock availability, product details, or delivery terms.
- Customer responses should be tested before enabling fully automatic sending.
- Customer information should be handled securely, and credentials must never be committed to the repository.
- The workflow should be evaluated against real sales outcomes to improve classification accuracy.

## Future Improvements

Potential enhancements include:

- Integrating a CRM or Google Sheets to store qualified leads.
- Connecting the workflow to WhatsApp, email, or a website chat system.
- Adding automatic sales-team notifications for HOT leads.
- Using product catalogue and inventory data to provide more accurate answers.
- Tracking conversion rates by lead score.
- Adding error handling, retries, and a fallback response when AI output is invalid.
- Introducing a review step for uncertain classifications.

## Getting Started

1. Set up an n8n instance.
2. Configure Google Gemini credentials in n8n.
3. Create or import the workflow.
4. Configure the customer inquiry trigger and map the incoming data fields.
5. Add the AI prompt to the Gemini node.
6. Configure the Code node to parse the AI output.
7. Add the IF node to route leads based on `lead_status`.
8. Test the workflow with sample inquiries before deploying it.

## Project Status

**Status:** Initial implementation completed; further testing and integrations can be added.

## Author

Created as a practical project exploring AI-powered workflow automation, lead qualification, and business process optimization using n8n and Google Gemini.
