# AI Crop Compensation System - Design Document

## System Overview

This is an AI-powered platform that helps farmers in Andhra Pradesh access government compensation schemes when their crops are damaged by climate events. The system uses real-time climate data, intelligent chatbot assistance, and multilingual support to make government schemes accessible to all farmers.

---

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend Layer                        │
│  (React/Next.js - Responsive Web App)                       │
│  - Multilingual UI (Telugu, English, Hindi)                 │
│  - AI Chatbot Interface                                      │
│  - Voice Input/Output                                        │
│  - Location Selection & Maps                                 │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                      API Gateway Layer                       │
│  (Node.js/Express or Python/FastAPI)                        │
│  - RESTful APIs                                              │
│  - Authentication & Authorization                            │
│  - Request Routing                                           │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────┬──────────────────┬──────────────────────┐
│   Core Services  │  AI Services     │  External APIs       │
│                  │                  │                      │
│ • Location Svc   │ • Chatbot Engine │ • NASA POWER API    │
│ • Damage Detect  │ • NLP Processing │ • Google Translate  │
│ • Scheme Match   │ • Voice STT/TTS  │ • Google Speech     │
│ • Compensation   │ • LLM Integration│                      │
└──────────────────┴──────────────────┴──────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│                       Database Layer                         │
│  (MySQL/PostgreSQL)                                          │
│  - Location Data (Districts, Mandals, Villages)             │
│  - Farmer Profiles                                           │
│  - Scheme Rules & Translations                               │
│  - Chat History                                              │
│  - Climate Cache (optional)                                  │
└─────────────────────────────────────────────────────────────┘
```

---

## Component Design

### 1. Frontend Components

#### Main Pages/Views:

**Home Page**
- Language selector (Telugu/English/Hindi)
- Quick access to chatbot
- Location selection dropdown
- Current alerts/notifications

**Chatbot Interface**
- Chat window with message history
- Text input with send button
- Voice input button (microphone icon)
- Voice output toggle (speaker icon)
- Quick action buttons ("Check Eligibility", "Calculate Compensation")
- Language indicator

**Dashboard**
- Farmer profile summary
- Current climate status for their village
- Active alerts (drought/flood warnings)
- Eligible schemes list
- Compensation calculator

**Scheme Explorer**
- List of all government schemes
- Filter by eligibility
- Detailed scheme information
- Application guidance

**Profile Page**
- Farmer details (name, phone, village)
- Land information (size, crops)
- Edit profile option

#### UI Components:

```javascript
// Component Structure
src/
├── components/
│   ├── Chatbot/
│   │   ├── ChatWindow.jsx
│   │   ├── MessageBubble.jsx
│   │   ├── VoiceInput.jsx
│   │   ├── VoiceOutput.jsx
│   │   └── QuickActions.jsx
│   ├── Location/
│   │   ├── DistrictSelector.jsx
│   │   ├── MandalSelector.jsx
│   │   └── VillageSelector.jsx
│   ├── Climate/
│   │   ├── WeatherCard.jsx
│   │   ├── RainfallChart.jsx
│   │   └── AlertBanner.jsx
│   ├── Schemes/
│   │   ├── SchemeCard.jsx
│   │   ├── SchemeDetails.jsx
│   │   └── EligibilityChecker.jsx
│   └── Common/
│       ├── LanguageSwitcher.jsx
│       ├── Header.jsx
│       └── Footer.jsx
```

### 2. Backend Services

#### Core Services:

**Location Service**
```python
class LocationService:
    def get_districts(self):
        # Return list of districts
        
    def get_mandals(self, district_id):
        # Return mandals for a district
        
    def get_villages(self, mandal_id):
        # Return villages for a mandal
        
    def get_village_coordinates(self, village_id):
        # Return lat/lon for climate API
```

**Climate Service**
```python
class ClimateService:
    def fetch_current_data(self, village_id):
        # Get lat/lon from database
        # Call NASA POWER API
        # Return rainfall, temperature
        
    def detect_damage(self, village_id):
        # Analyze climate data
        # Check against thresholds
        # Return damage type and severity
        
    def get_historical_trends(self, village_id, months=6):
        # Fetch historical climate data
        # Return trends for visualization
```

**Scheme Service**
```python
class SchemeService:
    def get_all_schemes(self, language='en'):
        # Return all schemes in specified language
        
    def match_schemes(self, farmer_profile):
        # Check eligibility for each scheme
        # Return matched schemes with relevance score
        
    def calculate_compensation(self, farmer_profile, damage_info):
        # Calculate compensation from each scheme
        # Return total amount and breakdown
```

**Chatbot Service**
```python
class ChatbotService:
    def __init__(self):
        self.llm_client = OpenAI()  # or Gemini
        self.translator = TranslationService()
        
    def process_message(self, message, farmer_profile, language):
        # Detect intent
        intent = self.detect_intent(message)
        
        # Handle structured queries
        if intent == 'eligibility_check':
            return self.check_eligibility(farmer_profile)
        elif intent == 'compensation_query':
            return self.calculate_compensation(farmer_profile)
        
        # Handle conversational queries with LLM
        else:
            context = self.build_context(farmer_profile)
            response = self.llm_client.chat(message, context)
            
            # Translate if needed
            if language != 'en':
                response = self.translator.translate(response, language)
                
            return response
    
    def detect_intent(self, message):
        # Simple keyword matching or use NLP
        keywords = {
            'eligibility': ['eligible', 'qualify', 'అర్హత'],
            'compensation': ['compensation', 'amount', 'money', 'పరిహారం'],
            'application': ['apply', 'how to', 'process', 'దరఖాస్తు']
        }
        # Return detected intent
```

### 3. AI/ML Components

#### Chatbot Architecture

```
User Input (Text/Voice)
        ↓
[Language Detection]
        ↓
[Intent Classification]
        ↓
    ┌───┴───┐
    │       │
Structured  Conversational
Query       Query
    │       │
    │       ↓
    │   [Context Builder]
    │       ↓
    │   [LLM API Call]
    │       ↓
    └───┬───┘
        ↓
[Translation (if needed)]
        ↓
[Response Formatting]
        ↓
Output (Text/Voice)
```

#### Intent Classification

```python
# Simple rule-based approach for hackathon
def classify_intent(message):
    message_lower = message.lower()
    
    # Eligibility check
    if any(word in message_lower for word in ['eligible', 'qualify', 'అర్హత', 'అర్హుడు']):
        return 'eligibility_check'
    
    # Compensation query
    if any(word in message_lower for word in ['compensation', 'amount', 'how much', 'పరిహారం', 'ఎంత']):
        return 'compensation_query'
    
    # Application process
    if any(word in message_lower for word in ['apply', 'application', 'how to', 'దరఖాస్తు', 'ఎలా']):
        return 'application_process'
    
    # Climate info
    if any(word in message_lower for word in ['rainfall', 'weather', 'temperature', 'వర్షపాతం']):
        return 'climate_info'
    
    # Default to conversational
    return 'conversational'
```

#### LLM Integration

```python
# Using OpenAI API (or Gemini)
def get_llm_response(user_message, farmer_context):
    system_prompt = f"""
    You are a helpful assistant for farmers in Andhra Pradesh.
    You help them understand government compensation schemes.
    
    Farmer Context:
    - Village: {farmer_context['village']}
    - Land Size: {farmer_context['land_size']} acres
    - Crops: {farmer_context['crops']}
    
    Available Schemes:
    - PM Fasal Bima Yojana (Crop Insurance)
    - PM-KISAN (Direct Income Support)
    - PM Krishi Sinchayee Yojana (Irrigation)
    
    Respond in simple, clear language. Be helpful and empathetic.
    """
    
    response = openai.ChatCompletion.create(
        model="gpt-4",
        messages=[
            {"role": "system", "content": system_prompt},
            {"role": "user", "content": user_message}
        ],
        temperature=0.7,
        max_tokens=200
    )
    
    return response.choices[0].message.content
```

### 4. Multilingual System

#### Translation Service

```python
class TranslationService:
    def __init__(self):
        self.google_translate = GoogleTranslateAPI()
        self.cache = {}  # Cache translations
        
    def translate(self, text, target_language):
        # Check cache first
        cache_key = f"{text}_{target_language}"
        if cache_key in self.cache:
            return self.cache[cache_key]
        
        # Translate
        if target_language == 'te':
            translated = self.google_translate.translate(text, dest='te')
        elif target_language == 'hi':
            translated = self.google_translate.translate(text, dest='hi')
        else:
            translated = text
        
        # Cache and return
        self.cache[cache_key] = translated
        return translated
    
    def get_scheme_content(self, scheme_id, language):
        # Get pre-translated scheme content from database
        query = """
            SELECT name, description, eligibility, how_to_apply
            FROM schemes_i18n
            WHERE scheme_id = ? AND language = ?
        """
        return db.execute(query, [scheme_id, language])
```

#### Voice Integration

```javascript
// Frontend - Speech-to-Text
class VoiceInput {
  constructor(language) {
    this.recognition = new webkitSpeechRecognition();
    this.recognition.lang = this.getLanguageCode(language);
    this.recognition.continuous = false;
    this.recognition.interimResults = false;
  }
  
  getLanguageCode(language) {
    const codes = {
      'telugu': 'te-IN',
      'english': 'en-IN',
      'hindi': 'hi-IN'
    };
    return codes[language] || 'en-IN';
  }
  
  start(onResult) {
    this.recognition.onresult = (event) => {
      const transcript = event.results[0][0].transcript;
      onResult(transcript);
    };
    this.recognition.start();
  }
}

// Frontend - Text-to-Speech
class VoiceOutput {
  speak(text, language) {
    const utterance = new SpeechSynthesisUtterance(text);
    utterance.lang = this.getLanguageCode(language);
    utterance.rate = 0.9;  // Slightly slower for clarity
    speechSynthesis.speak(utterance);
  }
  
  getLanguageCode(language) {
    const codes = {
      'telugu': 'te-IN',
      'english': 'en-IN',
      'hindi': 'hi-IN'
    };
    return codes[language] || 'en-IN';
  }
}
```

---

## Database Design

### Schema

```sql
-- Districts
CREATE TABLE districts (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    state VARCHAR(50) DEFAULT 'Andhra Pradesh'
);

-- Mandals
CREATE TABLE mandals (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    district_id INT NOT NULL,
    FOREIGN KEY (district_id) REFERENCES districts(id)
);

-- Villages
CREATE TABLE villages (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    code VARCHAR(20),
    mandal_id INT NOT NULL,
    latitude DECIMAL(10, 8) NOT NULL,
    longitude DECIMAL(11, 8) NOT NULL,
    FOREIGN KEY (mandal_id) REFERENCES mandals(id),
    INDEX idx_coordinates (latitude, longitude)
);

-- Farmers
CREATE TABLE farmers (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100) NOT NULL,
    phone VARCHAR(15) UNIQUE NOT NULL,
    village_id INT NOT NULL,
    land_size DECIMAL(10, 2),
    crops JSON,
    ownership_type ENUM('owned', 'tenant') DEFAULT 'owned',
    preferred_language VARCHAR(10) DEFAULT 'te',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (village_id) REFERENCES villages(id)
);

-- Schemes (Base table)
CREATE TABLE schemes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    scheme_code VARCHAR(50) UNIQUE NOT NULL,
    website VARCHAR(200),
    eligibility_rules JSON,
    compensation_rules JSON,
    active BOOLEAN DEFAULT TRUE
);

-- Schemes Translations
CREATE TABLE schemes_i18n (
    id INT PRIMARY KEY AUTO_INCREMENT,
    scheme_id INT NOT NULL,
    language VARCHAR(10) NOT NULL,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    eligibility TEXT,
    how_to_apply TEXT,
    FOREIGN KEY (scheme_id) REFERENCES schemes(id),
    UNIQUE KEY unique_scheme_lang (scheme_id, language)
);

-- Chat History
CREATE TABLE chat_history (
    id INT PRIMARY KEY AUTO_INCREMENT,
    farmer_id INT NOT NULL,
    message TEXT NOT NULL,
    response TEXT NOT NULL,
    language VARCHAR(10),
    intent VARCHAR(50),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (farmer_id) REFERENCES farmers(id),
    INDEX idx_farmer_date (farmer_id, created_at)
);

-- Climate Cache (optional - for performance)
CREATE TABLE climate_cache (
    id INT PRIMARY KEY AUTO_INCREMENT,
    village_id INT NOT NULL,
    date DATE NOT NULL,
    rainfall DECIMAL(10, 2),
    temperature DECIMAL(5, 2),
    fetched_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (village_id) REFERENCES villages(id),
    UNIQUE KEY unique_village_date (village_id, date)
);

-- Damage Alerts
CREATE TABLE damage_alerts (
    id INT PRIMARY KEY AUTO_INCREMENT,
    village_id INT NOT NULL,
    damage_type ENUM('drought', 'flood', 'heat_stress') NOT NULL,
    severity ENUM('low', 'medium', 'high') NOT NULL,
    detected_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    resolved BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (village_id) REFERENCES villages(id),
    INDEX idx_village_active (village_id, resolved)
);
```

---

## API Design

### REST API Endpoints

#### Location APIs
```
GET  /api/districts
     Response: [{ id, name }]

GET  /api/mandals?district_id={id}
     Response: [{ id, name, district_id }]

GET  /api/villages?mandal_id={id}
     Response: [{ id, name, code, latitude, longitude }]
```

#### Climate APIs
```
GET  /api/climate/current?village_id={id}
     Response: { rainfall, temperature, date, source }

GET  /api/climate/trends?village_id={id}&months=6
     Response: [{ month, rainfall, temperature }]

GET  /api/damage/check?village_id={id}
     Response: { has_damage, type, severity, description }
```

#### Scheme APIs
```
GET  /api/schemes?language={lang}
     Response: [{ id, name, description, website }]

POST /api/schemes/match
     Body: { farmer_id }
     Response: [{ scheme, eligibility_score, estimated_compensation }]

POST /api/compensation/calculate
     Body: { farmer_id, damage_type, severity }
     Response: { total, breakdown: [{ scheme, amount }] }
```

#### Chatbot APIs
```
POST /api/chat/message
     Body: { farmer_id, message, language }
     Response: { response, intent, suggestions }

GET  /api/chat/history?farmer_id={id}&limit=20
     Response: [{ message, response, timestamp }]

POST /api/chat/voice
     Body: { farmer_id, audio_data, language }
     Response: { transcript, response, audio_response_url }
```

#### Farmer APIs
```
POST /api/farmers/register
     Body: { name, phone, village_id, land_size, crops, language }
     Response: { farmer_id, token }

GET  /api/farmers/{id}
     Response: { id, name, phone, village, land_size, crops }

PUT  /api/farmers/{id}
     Body: { land_size, crops, preferred_language }
     Response: { success, updated_fields }
```

#### Translation APIs
```
POST /api/translate
     Body: { text, target_language }
     Response: { translated_text, source_language }

GET  /api/languages
     Response: [{ code: 'te', name: 'Telugu' }, ...]
```

---

## Data Flow Examples

### Example 1: Farmer Asks About Eligibility via Chatbot

```
1. Farmer types: "నేను PM-KISAN కి అర్హుడినా?" (Am I eligible for PM-KISAN?)

2. Frontend sends to backend:
   POST /api/chat/message
   { farmer_id: 123, message: "నేను PM-KISAN కి అర్హుడినా?", language: "te" }

3. Backend processes:
   - Detect language: Telugu
   - Translate to English: "Am I eligible for PM-KISAN?"
   - Classify intent: eligibility_check
   - Get farmer profile from database
   - Check PM-KISAN eligibility rules
   - Generate response: "Yes, you are eligible..."
   - Translate back to Telugu

4. Backend responds:
   { 
     response: "అవును, మీరు PM-KISAN కి అర్హులు...",
     intent: "eligibility_check",
     suggestions: ["How much will I get?", "How to apply?"]
   }

5. Frontend displays response in chat
```

### Example 2: Automatic Damage Detection

```
1. Cron job runs daily:
   - For each village in database
   - Fetch climate data from NASA API
   - Check against thresholds

2. If damage detected:
   - Create alert in damage_alerts table
   - Get all farmers in that village
   - (Future: Send SMS/notification)

3. When farmer logs in:
   - Check for active alerts in their village
   - Display alert banner
   - Chatbot proactively suggests: "I noticed low rainfall in your village. Would you like to check compensation options?"
```

---

## Technology Stack Summary

**Frontend:**
- React.js with Next.js
- Tailwind CSS
- Web Speech API (for voice)
- Axios for API calls

**Backend:**
- Node.js with Express (or Python with FastAPI)
- JWT for authentication
- Axios/Requests for external APIs

**Database:**
- MySQL or PostgreSQL
- Redis (optional, for caching)

**AI/ML:**
- OpenAI GPT-4 or Google Gemini (chatbot)
- LangChain (optional, for orchestration)
- Google Translate API
- Google Speech-to-Text API
- Google Text-to-Speech API

**External APIs:**
- NASA POWER API (climate data)
- Google Maps API (optional, for maps)

**Deployment:**
- Docker containers
- Cloud hosting (AWS/Azure/GCP)
- CI/CD with GitHub Actions

---

## Security Considerations

1. **Authentication**: JWT tokens for API access
2. **Data Encryption**: HTTPS only, encrypt sensitive farmer data
3. **API Rate Limiting**: Prevent abuse of chatbot and external APIs
4. **Input Validation**: Sanitize all user inputs
5. **API Key Security**: Store API keys in environment variables

---

## Performance Optimization

1. **Caching**: Cache climate data for 24 hours
2. **Database Indexing**: Index on village_id, farmer_id, dates
3. **CDN**: Serve static assets via CDN
4. **Lazy Loading**: Load components on demand
5. **API Response Compression**: Gzip responses

---

## Scalability Plan

**Current (Hackathon):**
- 2 districts, ~100 villages
- 100 concurrent users
- Simple deployment

**Future (Production):**
- All 13 districts, 10,000+ villages
- 10,000+ concurrent users
- Microservices architecture
- Load balancing
- Database sharding by district
- Message queue for async tasks

---

## Testing Strategy

**Unit Tests:**
- Test each service independently
- Mock external API calls

**Integration Tests:**
- Test API endpoints
- Test database operations

**User Testing:**
- Test chatbot with real Telugu queries
- Test voice input/output
- Test on mobile devices

**Demo Preparation:**
- Prepare sample farmer profiles
- Pre-cache some climate data
- Have backup responses if APIs fail

---

## Success Criteria

 Chatbot responds correctly to 10+ common queries  
 Voice input/output works in Telugu and English  
 Climate data fetched successfully from NASA API  
 Scheme matching works accurately  
 Compensation calculator gives correct amounts  
 UI is responsive and works on mobile  
 System handles 100 concurrent users  
 All features work in Telugu, English, and Hindi  

---

## Future Enhancements

- WhatsApp bot integration
- SMS notifications for alerts
- Mobile app (React Native)
- Offline mode
- Integration with government portals for direct application
- Satellite imagery for crop damage assessment
- Predictive analytics for crop yield
- Blockchain for transparency in compensation distribution

---

This design provides a solid foundation for a 14-hour hackathon while being scalable for real-world deployment.

