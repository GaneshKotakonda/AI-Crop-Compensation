# AI Crop Compensation System - Requirements Document

## The Problem We're Solving

Farmers in Andhra Pradesh face a real challenge. When drought hits or floods destroy their crops, they struggle to find out what help is available. They don't know which government schemes they qualify for, how much compensation they can get, or how to apply. The information exists but it's scattered across different websites, buried in PDFs, or only available in English.

We're building an AI-powered system that changes this. It automatically detects crop damage using climate data, matches farmers with the right schemes, calculates their compensation, and guides them through the application process - all in their local language.

---

## What This System Does

### Core Functionality

1. **AI Chatbot Interface**
   - Conversational AI assistant that understands farmer queries in Telugu and English
   - Natural language processing to answer questions about schemes, compensation, and eligibility
   - Voice-enabled for farmers with low literacy
   - Guides farmers step-by-step through the entire process
   - Available 24/7 without human intervention

2. **Automatic Damage Detection**
   - Monitors rainfall, temperature, and groundwater levels for villages across Andhra Pradesh
   - Detects drought, flood, and other climate-related crop damage
   - Alerts farmers when their crops are at risk

3. **Smart Scheme Matching**
   - Analyzes farmer profile (land size, crop type, location)
   - Matches them with eligible government schemes
   - Explains benefits in simple language (Telugu/English/Hindi)

4. **Compensation Calculator**
   - Calculates exact compensation amount based on damage severity
   - Shows breakdown of benefits from different schemes
   - Provides realistic timeline for receiving funds

5. **Application Assistant**
   - Guides farmers step-by-step through the application process
   - Lists required documents
   - Provides links to online portals or nearest help centers

6. **Multilingual Experience**
   - Complete system available in Telugu, English, and Hindi
   - Voice interaction for accessibility
   - Text-to-speech for reading scheme details
   - Speech-to-text for voice queries

---

## Data Requirements

### 1. Location Data (Administrative Hierarchy)

**What we need:**
- District names
- Mandal names under each district
- Village names under each mandal
- Village codes (official census codes)
- Latitude and longitude for each village

**Where to get it:**
- Census of India 2011 village directory
- Data.gov.in (search: "Andhra Pradesh village list")
- Format: CSV/Excel files

**Scope for hackathon:**
- Focus on 2 districts: Anantapur and Kurnool
- 3-4 mandals per district
- 10-15 villages per mandal
- Total: ~60-120 villages

**Database structure:**
```
districts (id, name, state)
mandals (id, name, district_id)
villages (id, name, code, mandal_id, latitude, longitude)
```

### 2. Climate Data (Real-time and Historical)

**What we need:**
- Monthly rainfall (mm)
- Average temperature (°C)
- Solar radiation (optional)
- Historical data for pattern analysis

**Where to get it:**
- NASA POWER API (primary source - free, real-time)
- IMD (Indian Meteorological Department) for historical data

**API Details:**
```
Endpoint: https://power.larc.nasa.gov/api/temporal/monthly/point
Parameters: PRECTOTCORR (rainfall), T2M (temperature)
Input: latitude, longitude, date range
Output: JSON with monthly climate data
```

**Implementation approach:**
- Don't store all climate data in database
- Fetch real-time data from NASA API when needed
- Store only critical thresholds and alerts

**Thresholds for damage detection:**
- Drought: Rainfall < 50mm for 2 consecutive months
- Flood: Rainfall > 300mm in a single month
- Heat stress: Temperature > 40°C for extended period

### 3. Groundwater Data

**What we need:**
- Groundwater level (meters below ground)
- Water quality indicators
- Seasonal variations

**Where to get it:**
- Central Ground Water Board (CGWB) reports
- Format: PDF reports, Excel tables

**For hackathon:**
- Use sample data (10-20 data points per district)
- Focus on showing the concept, not comprehensive coverage
- Manually extract from CGWB reports

**Database structure:**
```
groundwater_data (id, village_id, measurement_date, water_level, quality)
```

### 4. Government Scheme Data

**What we need:**
- Scheme names
- Eligibility criteria (farmer type, land size, crop type)
- Compensation rules (how much, for what damage)
- Application process
- Official website links

**Key schemes to include:**

1. **PM Fasal Bima Yojana (Crop Insurance)**
   - Website: pmfby.gov.in
   - Coverage: All notified crops
   - Compensation: Up to 90% of sum insured
   - Covers: Drought, flood, pest attacks, natural calamities

2. **PM-KISAN (Direct Income Support)**
   - Website: pmkisan.gov.in
   - Eligibility: All landholding farmers
   - Benefit: ₹6000 per year (₹2000 × 3 installments)

3. **Pradhan Mantri Krishi Sinchayee Yojana**
   - Website: pmksy.gov.in
   - Focus: Irrigation, water conservation
   - Benefits: Subsidies for drip irrigation, sprinklers

4. **State-level schemes (Andhra Pradesh)**
   - YSR Rythu Bharosa
   - Crop insurance schemes
   - Disaster relief funds

**Implementation approach:**
- Hardcode scheme rules in JSON format
- No API needed - manually extract from official websites
- Update periodically (schemes don't change frequently)

**Database structure:**
```
schemes (id, name, description, eligibility_json, compensation_json, website, language)
```

### 5. Crop Data

**What we need:**
- Common crops in Andhra Pradesh
- Growing seasons
- Water requirements
- Typical yield per acre

**Major crops to include:**
- Paddy (Rice)
- Cotton
- Groundnut
- Maize
- Sugarcane
- Chillies
- Turmeric

**Database structure:**
```
crops (id, name, season, water_requirement, typical_yield)
```

### 6. Farmer Data (User Input)

**What farmers will provide:**
- Name, phone number
- Village location
- Land size (acres)
- Crops grown
- Land ownership type (owned/tenant)
- Aadhaar number (optional, for verification)

**Database structure:**
```
farmers (id, name, phone, village_id, land_size, crops, ownership_type)
```

---

## Functional Requirements

### FR1: User Registration
- Farmers can register with basic details
- Support for Telugu and English
- Mobile-friendly interface
- SMS verification (optional)

### FR2: Location Selection
- Dropdown: District → Mandal → Village
- Auto-detect location using GPS (if on mobile)
- Show village on map

### FR3: Climate Monitoring
- Fetch real-time climate data from NASA API
- Display current rainfall and temperature
- Show historical trends (last 6 months)
- Alert if conditions indicate crop damage

### FR4: Damage Assessment
- AI analyzes climate data against thresholds
- Detects: drought, flood, heat stress
- Calculates damage severity (low/medium/high)
- Shows affected crops

### FR5: Scheme Recommendation
- Match farmer profile with scheme eligibility
- Rank schemes by relevance
- Show estimated compensation amount
- Explain in simple language

### FR6: Compensation Calculator
- Input: damage type, severity, land size, crop
- Output: estimated compensation from each scheme
- Total compensation amount
- Expected timeline

### FR7: Application Guidance
- Step-by-step application process
- Required documents checklist
- Links to online portals
- Nearest help center locations

### FR8: AI Chatbot Assistant
- Conversational interface for farmers
- Natural language understanding in Telugu and English
- Answer questions about schemes, eligibility, and compensation
- Guide through application process via chat
- Voice input and output support
- Context-aware responses based on farmer profile
- 24/7 availability

**Chatbot Capabilities:**
- "What schemes am I eligible for?"
- "How much compensation can I get for drought?"
- "What documents do I need?"
- "Where is the nearest help center?"
- "Explain PM Fasal Bima Yojana in simple words"
- "Check my village's rainfall this month"

### FR9: Multilingual Support
- Full interface in Telugu, English, and Hindi
- Scheme descriptions translated to local languages
- Voice input/output in Telugu and English
- Text-to-speech for low literacy users
- Language switching anytime
- Regional dialect support for Telugu

### FR10: Admin Dashboard
- View all registered farmers
- Monitor climate alerts
- Track application status
- Generate reports

---

## Non-Functional Requirements

### NFR1: Performance
- API response time < 2 seconds
- Support 1000+ concurrent users
- Climate data refresh every 24 hours

### NFR2: Scalability
- Start with 2 districts (60-120 villages)
- Architecture should support scaling to all 13 districts
- Database design for 10,000+ villages

### NFR3: Reliability
- 99% uptime
- Fallback if NASA API is down
- Data backup daily

### NFR4: Security
- Encrypt farmer personal data
- Secure API endpoints
- HTTPS only

### NFR5: Usability
- Simple, intuitive interface
- Works on low-end smartphones
- Minimal data usage
- Offline mode for basic features (optional)

### NFR6: Accessibility
- Support for low literacy users
- Voice guidance
- Large fonts, clear icons
- Works on 2G/3G networks

---

## AI Chatbot Requirements

### Chatbot Personality and Tone
- Friendly, helpful, and patient
- Uses simple language (avoid technical jargon)
- Empathetic to farmer's situation
- Culturally aware and respectful

### Chatbot Knowledge Base

**Must be able to answer:**
1. Scheme eligibility questions
   - "Am I eligible for PM Fasal Bima Yojana?"
   - "What schemes are available for small farmers?"
   
2. Compensation queries
   - "How much will I get for drought damage?"
   - "What is the compensation for 2 acres of paddy?"
   
3. Application process
   - "How do I apply for PM-KISAN?"
   - "What documents do I need?"
   - "Where is the nearest CSC center?"
   
4. Climate information
   - "What is the rainfall in my village this month?"
   - "Is there a drought alert for my area?"
   
5. General guidance
   - "What is crop insurance?"
   - "When will I receive the money?"
   - "Can tenant farmers apply?"

### Chatbot Features

**Context Awareness:**
- Remember farmer's profile (location, crops, land size)
- Reference previous conversation
- Personalize responses based on farmer's situation

**Proactive Suggestions:**
- "Based on low rainfall in your village, you may be eligible for drought compensation"
- "Have you considered applying for PM-KISAN? You seem eligible"

**Fallback Handling:**
- If chatbot doesn't understand, offer options
- Escalate to human support (future feature)
- Provide relevant links or phone numbers

**Multi-turn Conversations:**
```
Farmer: "I want to apply for scheme"
Bot: "Sure! Which scheme are you interested in? We have PM Fasal Bima Yojana, PM-KISAN, and others."
Farmer: "The crop insurance one"
Bot: "Great! PM Fasal Bima Yojana provides crop insurance. Let me check if you're eligible..."
```

### Chatbot Implementation Options

**Option 1: Rule-based + LLM Hybrid (Recommended)**
- Use rules for structured queries (eligibility, calculation)
- Use LLM (GPT-4, Gemini) for natural conversation
- Combine for best accuracy and flexibility

**Option 2: Fine-tuned Model**
- Fine-tune open-source model (Llama, Mistral) on scheme data
- More control, but requires more effort
- Good for production, overkill for hackathon

**Option 3: RAG (Retrieval Augmented Generation)**
- Store scheme documents in vector database
- Retrieve relevant info and pass to LLM
- Good balance of accuracy and flexibility

**For Hackathon: Use Option 1**
```python
# Pseudo-code
def chatbot_response(user_message, farmer_profile):
    # Check if it's a structured query
    if is_eligibility_query(user_message):
        return check_eligibility(farmer_profile)
    
    elif is_compensation_query(user_message):
        return calculate_compensation(farmer_profile)
    
    # Otherwise, use LLM
    else:
        context = get_relevant_schemes(farmer_profile)
        prompt = f"Context: {context}\nFarmer: {user_message}\nAssistant:"
        return llm_api.generate(prompt)
```

### Chatbot Training Data

**Create a dataset of common queries:**
```json
[
  {
    "query": "నేను PM-KISAN కి అర్హుడినా?",
    "intent": "eligibility_check",
    "scheme": "PM-KISAN",
    "language": "telugu"
  },
  {
    "query": "How much compensation for drought?",
    "intent": "compensation_query",
    "damage_type": "drought",
    "language": "english"
  }
]
```

---

## Multilingual Requirements

### Supported Languages
1. **Telugu** (Primary - most farmers in AP)
2. **English** (Secondary - educated farmers, officials)
3. **Hindi** (Optional - migrant workers)

### Translation Scope

**What needs translation:**
- All UI text (buttons, labels, menus)
- Scheme names and descriptions
- Eligibility criteria
- Application instructions
- Error messages
- Chatbot responses

**What doesn't need translation:**
- Farmer names, village names
- Numbers, dates
- Official document names (Aadhaar, etc.)

### Translation Implementation

**Option 1: Google Translate API**
- Pros: Easy, supports 100+ languages, good quality
- Cons: Costs money (but cheap), requires internet
- Best for: Quick implementation

**Option 2: Pre-translated Content**
- Manually translate all scheme content
- Store in database with language field
- Pros: Free, accurate, works offline
- Cons: Manual work, hard to update
- Best for: Static content like scheme descriptions

**Option 3: Hybrid Approach (Recommended)**
- Pre-translate important content (schemes, instructions)
- Use API for dynamic content (chatbot responses)
- Best of both worlds

### Database Schema for Multilingual

```sql
CREATE TABLE schemes_i18n (
    id INT PRIMARY KEY AUTO_INCREMENT,
    scheme_id INT,
    language VARCHAR(10),
    name VARCHAR(200),
    description TEXT,
    eligibility TEXT,
    how_to_apply TEXT,
    FOREIGN KEY (scheme_id) REFERENCES schemes(id)
);
```

**Example data:**
```sql
INSERT INTO schemes_i18n VALUES
(1, 1, 'en', 'PM Fasal Bima Yojana', 'Crop insurance scheme...', ...),
(2, 1, 'te', 'పిఎం ఫసల్ బీమా యోజన', 'పంట బీమా పథకం...', ...),
(3, 1, 'hi', 'पीएम फसल बीमा योजना', 'फसल बीमा योजना...', ...);
```

### Voice Features

**Speech-to-Text (STT):**
- Google Speech-to-Text API
- Supports Telugu, English, Hindi
- Real-time transcription
- Handle background noise (rural areas)

**Text-to-Speech (TTS):**
- Google Text-to-Speech API
- Natural-sounding voices
- Adjustable speed for clarity
- Male/female voice options

**Implementation:**
```javascript
// Frontend - Voice Input
const startVoiceInput = () => {
  const recognition = new webkitSpeechRecognition();
  recognition.lang = 'te-IN'; // Telugu
  recognition.onresult = (event) => {
    const transcript = event.results[0][0].transcript;
    sendToChatbot(transcript);
  };
  recognition.start();
};

// Frontend - Voice Output
const speakResponse = (text, language) => {
  const utterance = new SpeechSynthesisUtterance(text);
  utterance.lang = language === 'telugu' ? 'te-IN' : 'en-US';
  speechSynthesis.speak(utterance);
};
```

### Language Detection
- Auto-detect language from user input
- Allow manual language switching
- Remember user's preferred language

### Cultural Considerations
- Use respectful terms (గారు in Telugu)
- Avoid complex English words
- Use local units (acres, not hectares)
- Reference local festivals, seasons
- Respect regional dialects

---

## Technical Requirements

### Tech Stack

**Frontend:**
- React.js or Next.js
- Responsive design (mobile-first)
- Tailwind CSS for styling

**Backend:**
- Node.js with Express or Python with Flask/FastAPI
- RESTful API architecture

**Database:**
- MySQL or PostgreSQL for structured data
- Redis for caching (optional)

**AI/ML:**
- Python for data analysis
- OpenAI API / Google Gemini / Anthropic Claude for chatbot
- LangChain for chatbot orchestration
- Google Translate API or custom translation model
- Speech-to-text: Google Speech API or Whisper
- Text-to-speech: Google TTS or Azure Speech
- Simple rule-based system for damage detection
- Machine learning model (optional, for advanced prediction)

**APIs:**
- NASA POWER API for climate data
- Google Maps API for location (optional)
- OpenAI/Gemini API for conversational AI
- Google Translate API for multilingual support
- Google Speech-to-Text API
- Google Text-to-Speech API

**Deployment:**
- Cloud hosting (AWS, Azure, or Google Cloud)
- Docker containers
- CI/CD pipeline

### API Endpoints Needed

```
GET  /api/districts - List all districts
GET  /api/mandals/:district_id - List mandals in a district
GET  /api/villages/:mandal_id - List villages in a mandal
GET  /api/climate/:village_id - Get climate data for a village
POST /api/assess-damage - Assess crop damage based on climate
GET  /api/schemes - List all schemes
POST /api/match-schemes - Match farmer with eligible schemes
POST /api/calculate-compensation - Calculate compensation amount
POST /api/farmers - Register a new farmer
GET  /api/farmers/:id - Get farmer details

# Chatbot Endpoints
POST /api/chat - Send message to chatbot
GET  /api/chat/history/:farmer_id - Get chat history
POST /api/chat/voice - Voice input to chatbot
GET  /api/chat/voice/:message_id - Get voice response

# Translation Endpoints
POST /api/translate - Translate text between languages
GET  /api/languages - List supported languages
```

---

## Success Metrics

### For Hackathon Demo:
- System works for 2 districts (Anantapur, Kurnool)
- Real-time climate data fetched successfully
- Accurate scheme matching for sample farmer profiles
- AI chatbot responds correctly to 10+ common queries
- Chatbot works in both Telugu and English
- Voice input/output functional
- Clean, working UI in all supported languages
- Complete end-to-end flow demonstrated with chatbot interaction

### For Real-world Impact:
- 1000+ farmers registered in first month
- 80%+ accuracy in damage detection
- Average time to find schemes: < 2 minutes
- 50%+ increase in scheme awareness among farmers
- Chatbot handles 90%+ of queries without human intervention
- Support for 3+ languages
- 70%+ user satisfaction with chatbot responses

---

## Constraints and Assumptions

### Constraints:
- Limited to Andhra Pradesh (for now)
- Depends on NASA API availability
- Scheme data must be manually updated
- No real-time satellite imagery (too expensive)

### Assumptions:
- Farmers have basic smartphone access
- Internet connectivity available (even if slow)
- Government scheme rules remain relatively stable
- Climate data from NASA is accurate enough for our use case

---

## Out of Scope (For Hackathon)

- Real-time satellite imagery analysis
- Automated application submission to government portals
- Payment processing
- SMS/WhatsApp bot integration (focus on web chatbot first)
- Mobile app (focus on web app)
- Integration with government databases
- Blockchain for transparency
- Advanced ML models for yield prediction
- Multi-language OCR for document scanning
- Video call support with agricultural experts

These can be added in future iterations.

---

## Risk Mitigation

**Risk 1: NASA API downtime**
- Mitigation: Cache recent data, have fallback to IMD data

**Risk 2: Incomplete government data**
- Mitigation: Use sample data for demo, clearly document sources

**Risk 3: Complex scheme eligibility rules**
- Mitigation: Start with simple rules, add complexity later

**Risk 4: Language translation accuracy**
- Mitigation: Get native Telugu speaker to review translations, use professional translation for key content

**Risk 5: Chatbot giving wrong information**
- Mitigation: Use rule-based system for critical queries (eligibility, compensation), LLM only for general conversation

**Risk 6: Voice recognition in noisy environments**
- Mitigation: Provide text input as alternative, use noise cancellation in STT API

**Risk 7: Time constraints**
- Mitigation: Focus on core features first, polish later

---

## Summary

This system bridges the gap between farmers and government support by:
1. Using freely available government data and APIs
2. Applying AI to detect crop damage automatically
3. Matching farmers with the right schemes
4. Simplifying the application process
5. **Providing an intelligent chatbot that speaks farmers' language**
6. **Breaking language barriers with multilingual support and voice interaction**

The AI chatbot makes the system accessible to farmers with low literacy, while multilingual support ensures no farmer is left behind due to language barriers.

It's practical, scalable, and solves a real problem. Perfect for a hackathon and ready to scale for real-world impact.