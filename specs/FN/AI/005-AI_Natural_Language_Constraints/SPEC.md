# FN/AI/005 - AI Natural Language Constraints

**Feature Type:** AI/Natural Language Processing  
**Status:** Implementation  
**Priority:** High  
**Dependencies:** FN/ADM/RUL/002-Nurse_Patterns  

---

## 1. Overview

Transform Orto from a generic constraint calculator into an intelligent nurse rostering assistant by integrating AI-powered natural language constraint creation. Users can describe scheduling rules in plain English (e.g., "Amy cannot work Night shifts") instead of navigating complex forms.

## 2. Goals

- **Natural Language Interface:** Users type constraint descriptions instead of filling dropdown forms
- **AI Translation:** Automatically convert natural language → structured JSON constraint format
- **OpenRouter Integration:** Use OpenRouter API with configurable AI models (GPT-4o, Claude 3.5 Sonnet, etc.)
- **Backwards Compatibility:** Maintain existing Point and VerticalSum constraint forms alongside AI interface

## 3. User Stories

### US-005.1: Natural Language Constraint Creation
**As a** roster manager  
**I want to** describe scheduling rules in natural language  
**So that** I can create complex constraints without understanding technical terminology

**Acceptance Criteria:**
- User enters text: "Amy cannot work Night shifts on weekends"
- System displays parsed constraint preview before saving
- Constraint is saved and applied to roster generation
- User can edit or delete AI-generated constraints

### US-005.2: Multi-Constraint Translation
**As a** roster manager  
**I want to** describe multiple constraints in a single input  
**So that** I can efficiently define related scheduling rules

**Acceptance Criteria:**
- User enters: "Bob needs 2 days off after working 5 consecutive days. Alice prefers morning shifts."
- System generates 2 separate constraints
- Both constraints are validated and saved atomically

### US-005.3: Constraint Validation & Feedback
**As a** roster manager  
**I want to** see validation errors for impossible constraints  
**So that** I understand when rules conflict or are infeasible

**Acceptance Criteria:**
- System detects conflicting constraints (e.g., "Amy must work Day 1" + "Amy cannot work Day 1")
- User receives clear error message with suggested resolution
- Invalid constraints are not saved to database

---

## 4. Architecture

### 4.1 System Flow

```
┌─────────────────────────────────────────────────────────────┐
│ 1. USER INPUT (Frontend)                                   │
│    Component: ai-constraint-builder.tsx                    │
│    Input: "Amy cannot work Night shifts"                   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 2. SERVER ACTION                                            │
│    File: ai-constraint.action.ts                           │
│    Function: translateConstraintAction()                   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 3. AI TRANSLATOR SERVICE                                    │
│    File: ai-constraint-translator.service.ts               │
│    Method: translateToConstraint()                         │
│                                                             │
│    OpenRouter API Call:                                    │
│    - Endpoint: https://openrouter.ai/api/v1/chat/completions│
│    - Model: env.OPENROUTER_MODEL (e.g., gpt-4o)           │
│    - System Prompt: Few-shot constraint examples           │
│    - User Prompt: Natural language input                   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 4. PARSE & VALIDATE                                         │
│    - Extract JSON from AI response                         │
│    - Validate against Zod schemas                          │
│    - Check constraint type (point/vertical_sum/            │
│      horizontal_sum/sliding_window)                        │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 5. SAVE TO DATABASE                                         │
│    Prisma: Constraint.create()                             │
│    - name: Auto-generated from natural language            │
│    - type: Constraint type from AI                         │
│    - config: JSON constraint parameters                    │
│    - isActive: true                                        │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ 6. RETURN TO UI                                             │
│    Success: Display constraint card                        │
│    Error: Show validation message                          │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 Component Structure

```
web/src/
├── app/actions/
│   └── ai-constraint.action.ts          # Server action for AI translation
├── services/
│   └── ai-constraint-translator.service.ts  # OpenRouter integration
├── features/dashboard/components/
│   ├── ai-constraint-builder.tsx        # Chat-style input UI
│   └── add-constraint-form.tsx          # Updated with AI option
└── lib/validations/
    └── solver.ts                        # Extended constraint schemas
```

---

## 5. OpenRouter Integration

### 5.1 Environment Variables

```bash
# Required
OPENROUTER_API_KEY=sk-or-v1-xxxxx
OPENROUTER_MODEL=openai/gpt-4o

# Optional (for analytics)
OPENROUTER_SITE_URL=https://orto.ai
OPENROUTER_APP_NAME=Orto Nurse Scheduler
```

### 5.2 Supported Models

Users can configure any OpenRouter-compatible model:

- `openai/gpt-4o` - Best for complex constraint understanding
- `anthropic/claude-3.5-sonnet` - Strong reasoning, good for pattern detection
- `meta-llama/llama-3.1-70b-instruct` - Cost-effective alternative
- `google/gemini-pro-1.5` - Good multilingual support

### 5.3 API Configuration

```typescript
const openai = new OpenAI({
  baseURL: 'https://openrouter.ai/api/v1',
  apiKey: process.env.OPENROUTER_API_KEY,
  defaultHeaders: {
    'HTTP-Referer': process.env.OPENROUTER_SITE_URL,
    'X-Title': process.env.OPENROUTER_APP_NAME,
  },
})
```

---

## 6. Few-Shot Prompt Engineering

### 6.1 System Prompt Template

```
You are an expert constraint translator for nurse roster scheduling systems.

Your task is to convert natural language scheduling rules into structured JSON constraints.

## Constraint Types:

1. **point** - Force specific resource to specific state at specific time
   Format: {
     "type": "point",
     "resource": "STAFF_ID",
     "time_slot": 0-6,
     "state": 0 (Off) | 1 (Work) | 2 (OnCall)
   }

2. **vertical_sum** - Count resources in target state at time slot(s)
   Format: {
     "type": "vertical_sum",
     "time_slot": "ALL" | 0-6,
     "target_state": 1 | 2,
     "operator": ">=" | "<=" | "==",
     "value": number
   }

3. **horizontal_sum** - Count consecutive states for single resource
   Format: {
     "type": "horizontal_sum",
     "resource": "STAFF_ID",
     "time_slots": [0, 1, 2],
     "target_state": 1,
     "operator": "<=",
     "value": 3
   }

4. **sliding_window** - Pattern-based constraints with rest periods
   Format: {
     "type": "sliding_window",
     "resource": "STAFF_ID",
     "work_days": 5,
     "rest_days": 2,
     "target_state": 1
   }

## Examples:

User: "Amy cannot work on Monday"
Assistant: {
  "name": "Amy cannot work on Monday",
  "type": "point",
  "config": {
    "type": "point",
    "resource": "Amy",
    "time_slot": 0,
    "state": 0
  }
}

User: "At least 3 nurses must be working every day"
Assistant: {
  "name": "At least 3 nurses working every day",
  "type": "vertical_sum",
  "config": {
    "type": "vertical_sum",
    "time_slot": "ALL",
    "target_state": 1,
    "operator": ">=",
    "value": 3
  }
}

User: "Bob can work maximum 3 consecutive night shifts"
Assistant: {
  "name": "Bob max 3 consecutive nights",
  "type": "horizontal_sum",
  "config": {
    "type": "horizontal_sum",
    "resource": "Bob",
    "time_slots": [0, 1, 2, 3, 4, 5, 6],
    "target_state": 1,
    "operator": "<=",
    "value": 3
  }
}

User: "Alice needs 2 days off after working 5 consecutive days"
Assistant: {
  "name": "Alice 2 days off after 5 work days",
  "type": "sliding_window",
  "config": {
    "type": "sliding_window",
    "resource": "Alice",
    "work_days": 5,
    "rest_days": 2,
    "target_state": 1
  }
}

Return ONLY valid JSON. No markdown, no explanations.
```

---

## 7. API Endpoints

### 7.1 Server Actions

#### `translateConstraintAction(input: string)`
**Purpose:** Convert natural language to constraint JSON  
**Returns:** `{ success: boolean, constraint?: Constraint, error?: string }`

#### `saveAIConstraintAction(constraint: ConstraintInput)`
**Purpose:** Validate and save AI-generated constraint  
**Returns:** `{ success: boolean, constraintId?: string, error?: string }`

---

## 8. UI/UX Design

### 8.1 AI Constraint Builder Component

```tsx
<Card>
  <CardHeader>
    <CardTitle>Create Constraint with AI</CardTitle>
    <CardDescription>
      Describe scheduling rules in natural language
    </CardDescription>
  </CardHeader>
  
  <CardContent>
    <Textarea
      placeholder="Example: Amy cannot work Night shifts on weekends"
      value={input}
      onChange={(e) => setInput(e.target.value)}
    />
    
    {isLoading && (
      <div className="mt-4">
        <Skeleton className="h-20 w-full" />
      </div>
    )}
    
    {parsedConstraint && (
      <Card className="mt-4">
        <CardHeader>
          <CardTitle>Parsed Constraint</CardTitle>
        </CardHeader>
        <CardContent>
          <Badge>{parsedConstraint.type}</Badge>
          <pre>{JSON.stringify(parsedConstraint.config, null, 2)}</pre>
        </CardContent>
      </Card>
    )}
  </CardContent>
  
  <CardFooter>
    <Button onClick={handleTranslate} disabled={isLoading}>
      Generate Constraint
    </Button>
    {parsedConstraint && (
      <Button onClick={handleSave}>Save Constraint</Button>
    )}
  </CardFooter>
</Card>
```

### 8.2 Integration with Existing Form

Add third tab/option to `add-constraint-form.tsx`:

```tsx
<DropdownMenu>
  <DropdownMenuTrigger>Add Rule</DropdownMenuTrigger>
  <DropdownMenuContent>
    <DropdownMenuItem onClick={() => setFormType('point')}>
      Day Off (Point)
    </DropdownMenuItem>
    <DropdownMenuItem onClick={() => setFormType('vertical_sum')}>
      Min/Max Workers (Sum)
    </DropdownMenuItem>
    <DropdownMenuItem onClick={() => setFormType('ai')}>
      🤖 Natural Language (AI)
    </DropdownMenuItem>
  </DropdownMenuContent>
</DropdownMenu>
```

---

## 9. Error Handling

### 9.1 OpenRouter API Errors
- **401 Unauthorized:** Invalid API key → Show setup instructions
- **429 Rate Limit:** Exceeded quota → Suggest waiting or upgrading plan
- **500 Server Error:** OpenRouter down → Fallback to manual form

### 9.2 Translation Errors
- **Invalid JSON:** AI returned malformed response → Retry with clarified prompt
- **Unknown Constraint Type:** AI generated unsupported type → Show error message
- **Missing Required Fields:** Incomplete constraint → Request additional details

### 9.3 Validation Errors
- **Conflicting Constraints:** New constraint contradicts existing rule → Highlight conflict
- **Invalid Resource:** Staff member doesn't exist → Suggest existing staff
- **Out of Range:** Time slot > available days → Adjust to valid range

---

## 10. Testing Strategy

### 10.1 Unit Tests
- Test AI translator service with mock OpenRouter responses
- Validate constraint parsing for all 4 types
- Test error handling for malformed AI responses

### 10.2 Integration Tests
- End-to-end: Natural language input → Constraint saved → Roster generated
- Test with various phrasings of same constraint
- Verify constraint application in OR-Tools solver

### 10.3 Manual Testing Scenarios
1. "Amy cannot work on Monday" → Point constraint
2. "At least 3 nurses working every day" → Vertical sum
3. "Bob max 3 consecutive nights" → Horizontal sum
4. "Alice needs 2 days off after 5 work days" → Sliding window
5. "No night shifts on weekends" → Multiple point constraints

---

## 11. Performance Considerations

### 11.1 API Response Time
- Target: < 3 seconds for AI translation
- Implement loading states during API calls
- Cache common constraint patterns (future enhancement)

### 11.2 Cost Management
- Monitor OpenRouter API usage
- Set rate limits per user/organization
- Provide cost estimates in UI

---

## 12. Future Enhancements

### 12.1 Phase 2 Features
- **Multi-language support:** Translate constraints in Spanish, French, etc.
- **Constraint suggestions:** AI recommends constraints based on roster patterns
- **Conflict resolution:** AI suggests how to resolve infeasible constraint sets
- **Learning from history:** Fine-tune prompts based on user corrections

### 12.2 Advanced Features
- **Voice input:** Speak constraints instead of typing
- **Batch import:** Upload Excel/CSV with natural language rules
- **Smart templates:** Industry-standard constraint libraries (hospital, clinic, etc.)

---

## 13. Security & Privacy

### 13.1 Data Handling
- User input sent to OpenRouter (3rd party) - disclose in privacy policy
- Do not send PII or sensitive patient data
- Staff names/IDs are sent - ensure compliance with data regulations

### 13.2 API Key Security
- Store OPENROUTER_API_KEY in environment variables only
- Never expose in client-side code
- Rotate keys regularly
- Use server actions for all AI calls

---

## 14. Success Metrics

- **Adoption Rate:** % of users who use AI vs manual forms
- **Translation Accuracy:** % of AI-generated constraints accepted without edits
- **Time Savings:** Average time to create constraint (AI vs manual)
- **User Satisfaction:** NPS score for AI feature

---

## 15. Dependencies

- **External:** OpenRouter API (requires account + credit)
- **Internal:** FN/ADM/RUL/002-Nurse_Patterns (horizontal_sum, sliding_window implementations)
- **Libraries:** `openai` npm package (OpenRouter-compatible)

---

## 16. Rollout Plan

### Phase 1: Beta (Week 1)
- Deploy AI feature behind feature flag
- Invite 10 beta users for feedback
- Monitor API costs and response times

### Phase 2: General Availability (Week 2)
- Enable for all users
- Add onboarding tutorial for AI interface
- Collect usage analytics

### Phase 3: Optimization (Week 3+)
- Fine-tune prompts based on user data
- Add constraint suggestion feature
- Optimize for cost/performance

---

## 17. Appendix

### A. Constraint Type Comparison

| Type | Use Case | Complexity | AI Translation Accuracy |
|------|----------|------------|------------------------|
| Point | "Amy off on Monday" | Low | 95%+ |
| Vertical Sum | "3 nurses working" | Low | 90%+ |
| Horizontal Sum | "Max 3 consecutive" | Medium | 80%+ |
| Sliding Window | "5 on, 2 off pattern" | High | 70%+ |

### B. Example Natural Language Inputs

**Simple:**
- "Amy has Monday off"
- "At least 2 nurses every day"
- "Bob cannot work weekends"

**Medium:**
- "Amy cannot work more than 3 days in a row"
- "Exactly 2 nurses on-call on Friday"
- "No night shifts for Alice"

**Complex:**
- "After working 5 consecutive days, staff must have at least 2 days off"
- "No transitions from night shift to day shift the next day"
- "Bob prefers morning shifts and needs every Sunday off"

---

**Document Version:** 1.0  
**Last Updated:** November 21, 2025  
**Author:** System Architect  
**Status:** Implementation Ready
