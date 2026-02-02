# AgroSPM Requirements - Hackathon Edition

## What We're Building

A simple mobile app for Indian farmers to track their farming activities and get AI suggestions to save money and help the environment. Think "WhatsApp for farming data" with smart recommendations.

**Target Users**: Small farmers in rural India with basic smartphones
**Goal**: Help farmers save ₹10,000+ per year while being eco-friendly
**Tech**: AWS services + simple mobile app

## Key Terms (Keep It Simple!)

- **Farmer**: Our main user - someone growing crops who wants to save money
- **Admin**: Person managing the system (NGO worker, government official)
- **Public**: Anyone who wants to buy eco-friendly produce
- **Green Score**: Simple 1-10 rating of how eco-friendly a farm is
- **Eco Badge**: Digital certificate farmers can show to sell at higher prices
- **Smart Tips**: AI suggestions to save water, reduce waste, cut costs

## Requirements

### 1. Simple Login System

**What farmers want**: "I want to easily log into the app with my phone number"

**What it does**:
- Farmers register with phone number + OTP (no complex passwords!)
- Admins get special access to manage farmers
- Public users can view eco-friendly farms without logging in
- Works offline when network is poor

### 2. Easy Data Entry

**What farmers want**: "I want to quickly record what I used on my farm today"

**What it does**:
- Simple forms to enter: water used, electricity bill, waste generated, fertilizer used
- Voice input in local languages (Hindi, Tamil, etc.)
- Photo capture for receipts/bills
- Works offline, syncs when internet is available
- Takes less than 2 minutes to enter daily data

### 3. Green Score Calculator

**What farmers want**: "I want to know if I'm farming in an eco-friendly way"

**What it does**:
- Calculates simple 1-10 green score based on farmer's data
- Shows score breakdown: water efficiency, waste management, energy use
- Compares with other farmers in the same district
- Updates score weekly
- Shows improvement over time with simple graphs

### 4. Smart Money-Saving Tips

**What farmers want**: "I want suggestions to reduce my farming costs"

**What it does**:
- AI analyzes farmer's data and suggests cost-saving tips
- Examples: "Use drip irrigation to save ₹5000/month on water"
- Prioritizes tips by potential savings (highest first)
- Tracks which farmers tried the suggestions and if it worked
- Sends 1-2 tips per week via SMS/WhatsApp

### 5. Eco Badge System

**What farmers want**: "I want proof that I farm sustainably to sell at better prices"

**What it does**:
- Automatically gives eco badges to farmers with green score 7+
- Creates digital certificate with QR code
- Farmers can show badge to buyers for premium pricing
- Badge expires after 1 year, needs renewal
- Public can scan QR code to verify authenticity

### 6. Public Marketplace

**What buyers want**: "I want to find and buy eco-friendly produce"

**What it does**:
- Shows list of eco-certified farmers and their produce
- Displays farmer's green score and location
- Shows regional sustainability trends
- Allows QR code verification of eco badges
- No personal farmer data exposed (privacy protected)

### 7. Admin Dashboard

**What admins want**: "I want to monitor the system and help farmers"

**What it does**:
- View all farmers and their green scores
- Adjust scoring parameters based on local conditions
- Generate reports for government/NGOs
- Monitor system usage and data quality
- Send bulk messages to farmers

### 8. Reliable & Fast System

**What everyone wants**: "The app should work fast and never lose my data"

**What it does**:
- All data backed up on AWS cloud
- App works even with slow 2G internet
- Loads farmer dashboard in under 3 seconds
- Data never gets lost
- Works on basic Android phones (Android 6+)

### 9. Mobile-First Design

**What farmers want**: "Easy to use on my smartphone, even with poor internet"

**What it does**:
- Designed for small screens and touch input
- Works offline for data entry
- Simple Hindi/English interface
- Large buttons for easy tapping
- Voice input for illiterate farmers
- Uses minimal data (under 10MB per month)

### 10. Integration & Growth

**What the system needs**: "Connect with other farming systems and handle more users"

**What it does**:
- APIs to connect with government agriculture databases
- Integration with weather services for better recommendations
- Can handle 100,000+ farmers without slowing down
- Secure data sharing with agricultural research institutes
- Easy to add new features without breaking existing ones