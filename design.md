# AgroSPM Design - Hackathon Edition

## What We're Building (Simple Version)

A mobile app that helps Indian farmers save money and get eco-certified using cheap AI on AWS. Think of it as a "farming assistant" that runs on basic smartphones and costs almost nothing to operate.

**Budget**: Under $100/month for 10,000 farmers using AWS free tier + cheap services
**Timeline**: 48-hour hackathon MVP
**Target**: Rural farmers with Android phones and 2G internet

## AWS Architecture (Keep It Simple!)

```
[Farmer's Phone] 
    ↓ (2G/3G internet)
[AWS API Gateway] 
    ↓
[AWS Lambda Functions] (our main logic)
    ↓
[AWS DynamoDB] (farmer data) + [AWS S3] (photos/files)
    ↓
[AWS SageMaker] (cheap AI recommendations)
```

### Why AWS? (Perfect for Hackathons!)

- **AWS Lambda**: Pay only when farmers use the app (almost free!)
- **AWS DynamoDB**: NoSQL database that scales automatically
- **AWS S3**: Store farmer photos and documents cheaply
- **AWS SageMaker**: Pre-built AI models for recommendations
- **AWS SNS**: Send SMS notifications to farmers
- **AWS Cognito**: Handle login with phone numbers + OTP

## Hackathon-Friendly Tech Stack

### Frontend (Mobile App)
- **React Native**: One codebase for Android + iOS
- **Expo**: Deploy instantly without app store approval
- **AsyncStorage**: Store data offline when internet is poor
- **React Native Voice**: Voice input in Hindi/local languages

### Backend (AWS Serverless)
- **AWS Lambda**: Node.js functions (pay per use!)
- **AWS API Gateway**: Handle mobile app requests
- **AWS DynamoDB**: Store farmer data (NoSQL, auto-scaling)
- **AWS S3**: Store photos, receipts, certificates
- **AWS Cognito**: Phone number login with OTP
- **AWS SNS**: Send SMS tips to farmers

### AI/ML (Low-Cost)
- **AWS SageMaker**: Pre-trained models for recommendations
- **AWS Comprehend**: Analyze farmer feedback in local languages
- **Simple Python scripts**: Calculate green scores (no complex ML needed!)

### Cost Breakdown (Monthly for 10,000 farmers)
- Lambda: $5 (1M requests)
- DynamoDB: $10 (basic usage)
- S3: $5 (photo storage)
- SNS: $20 (SMS notifications)
- **Total: ~$40/month** 🎉

## Core Components (Simplified)

### 1. Farmer Data Tracker (AWS Lambda + DynamoDB)

**What it does**: Stores farmer's daily farming data
**AWS Services**: Lambda function + DynamoDB table

```javascript
// Simple data structure
{
  farmerId: "farmer123",
  date: "2024-01-15",
  waterUsed: 500, // liters
  electricityBill: 2000, // rupees
  wasteGenerated: 10, // kg
  fertilizer: 5 // kg
}
```

**API Endpoints**:
```
POST /api/data/add - Add daily farming data
GET /api/data/history - Get farmer's history
```

### 2. Green Score Calculator (Simple Python Logic)

**What it does**: Calculates 1-10 green score based on farmer's data
**Logic**: Simple weighted average (no complex AI needed!)

```python
def calculate_green_score(farmer_data):
    water_score = min(10, 1000 / farmer_data.water_used)
    waste_score = max(1, 10 - farmer_data.waste_generated / 5)
    energy_score = min(10, 5000 / farmer_data.electricity_bill)
    
    green_score = (water_score + waste_score + energy_score) / 3
    return round(green_score, 1)
```

### 3. AI Recommendation Engine (AWS SageMaker)

**What it does**: Gives simple money-saving tips
**How**: Pre-defined rules + basic pattern matching

```javascript
// Simple recommendation logic
const recommendations = [
  {
    condition: "high_water_usage",
    tip: "Try drip irrigation to save ₹3000/month on water bills",
    savings: 3000
  },
  {
    condition: "high_electricity",
    tip: "Use solar pump to reduce electricity costs by ₹2000/month",
    savings: 2000
  }
];
```

### 4. Eco Badge System (AWS S3 + Simple QR)

**What it does**: Creates digital certificates for good farmers
**How**: Generate QR code + store in S3

```javascript
// Badge creation
if (farmer.green_score >= 7) {
  const badge = {
    farmerId: farmer.id,
    score: farmer.green_score,
    validUntil: "2025-01-15",
    qrCode: generateQR(farmer.id)
  };
  // Store in S3 and send to farmer
}
```

## User Interfaces (Mobile-First)

### Farmer App (React Native)
- **Home Screen**: Today's green score + quick data entry
- **Data Entry**: Simple form with voice input
- **My Tips**: AI recommendations with potential savings
- **My Badge**: Eco certificate with QR code
- **History**: Simple graphs showing improvement

### Admin Web Dashboard (Simple React)
- **Farmer List**: All registered farmers + their scores
- **Analytics**: Regional green score averages
- **Messages**: Send bulk SMS tips to farmers
- **Settings**: Adjust scoring parameters

### Public Website (Static Site)
- **Find Eco Farmers**: Search by location
- **Verify Badge**: Scan QR code to verify authenticity
- **Regional Stats**: District-wise sustainability trends

## Simple Data Models (DynamoDB)

### Farmer Profile
```json
{
  "farmerId": "farmer_123",
  "phone": "+91-9876543210",
  "name": "राम कुमार",
  "village": "Rampur",
  "district": "Meerut",
  "state": "UP",
  "farmSize": 2.5,
  "crops": ["wheat", "rice"],
  "greenScore": 7.2,
  "ecoStatus": "certified"
}
```

### Daily Farm Data
```json
{
  "dataId": "data_456",
  "farmerId": "farmer_123",
  "date": "2024-01-15",
  "waterUsed": 500,
  "electricityBill": 2000,
  "wasteGenerated": 10,
  "fertilizerUsed": 5,
  "notes": "Used drip irrigation today"
}
```

### Recommendations
```json
{
  "tipId": "tip_789",
  "farmerId": "farmer_123",
  "category": "water",
  "message": "Try drip irrigation to save ₹3000/month",
  "potentialSavings": 3000,
  "status": "pending",
  "createdAt": "2024-01-15"
}
```

## Hackathon Implementation Plan

### Day 1 (24 hours) - Core MVP
1. **Setup AWS account + basic Lambda functions** (2 hours)
2. **Create React Native app with basic screens** (6 hours)
3. **Implement phone login with AWS Cognito** (4 hours)
4. **Build data entry form + DynamoDB storage** (6 hours)
5. **Create simple green score calculator** (4 hours)
6. **Test with dummy data** (2 hours)

### Day 2 (24 hours) - AI + Polish
1. **Add basic recommendation engine** (6 hours)
2. **Implement eco badge system with QR codes** (4 hours)
3. **Create admin dashboard** (6 hours)
4. **Add SMS notifications with AWS SNS** (3 hours)
5. **Build public verification website** (3 hours)
6. **Final testing + demo preparation** (2 hours)

### MVP Features (48-hour version)
- ✅ Phone number login
- ✅ Simple data entry (water, electricity, waste)
- ✅ Green score calculation (1-10)
- ✅ Basic AI tips ("Use less water to save money")
- ✅ Eco badge with QR code
- ✅ Admin dashboard
- ✅ Public farmer directory

### Post-Hackathon Enhancements
- Voice input in local languages
- Weather API integration
- Advanced ML recommendations
- Government database integration
- Offline sync capabilities

## Why This Design Works for Rural India

1. **Low Data Usage**: App uses <1MB per day
2. **Offline Support**: Core features work without internet
3. **Simple UI**: Large buttons, minimal text
4. **Voice Input**: For farmers who can't read/write
5. **SMS Backup**: Tips sent via SMS when app isn't used
6. **Local Language**: Hindi/regional language support
7. **Low-End Phone Support**: Works on Android 6+ devices

## Cost Optimization Tricks

1. **Use AWS Free Tier**: 1M Lambda requests/month free
2. **DynamoDB On-Demand**: Pay only for actual usage
3. **S3 Intelligent Tiering**: Automatically move old data to cheaper storage
4. **CloudFront**: Cache static content to reduce costs
5. **SMS Only for Important Tips**: Avoid unnecessary SMS costs
6. **Compress Images**: Reduce S3 storage costs

This design can handle 10,000 farmers for under $50/month! 🚀