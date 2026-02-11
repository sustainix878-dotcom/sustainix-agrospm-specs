# Design Document: AgroSPM

## 1. System Architecture Overview

### 1.1 High-Level Architecture

AgroSPM follows a modern cloud-native architecture built on AWS services with a React frontend, Node.js backend, and AI/ML capabilities powered by AWS SageMaker.

```
┌─────────────────────────────────────────────────────────────────┐
│                         Client Layer                             │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  React.js Web Application (Progressive Web App)          │  │
│  │  - Offline-first with Service Workers                    │  │
│  │  - Chart.js/Recharts for visualizations                  │  │
│  │  - Responsive rural-friendly UI                          │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              ↓ HTTPS/TLS 1.3
┌─────────────────────────────────────────────────────────────────┐
│                      API Gateway Layer                           │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  AWS API Gateway                                         │  │
│  │  - Rate limiting & throttling                            │  │
│  │  - API versioning                                        │  │
│  │  - Request/response transformation                       │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                    Application Layer                             │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Node.js + Express REST API                              │  │
│  │  - Authentication & Authorization                        │  │
│  │  - Business Logic                                        │  │
│  │  - Data Validation                                       │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      AI/ML Layer                                 │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  AWS SageMaker                                           │  │
│  │  - Predictive models (water/energy forecasting)          │  │
│  │  - Recommendation engine                                 │  │
│  │  - Anomaly detection                                     │  │
│  └──────────────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  AWS Lambda Functions                                    │  │
│  │  - Sustainability scoring                                │  │
│  │  - Carbon footprint calculation                          │  │
│  │  - Compliance evaluation                                 │  │
│  └──────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────────┐
│                      Data Layer                                  │
│  ┌────────────────────┐  ┌────────────────────┐                │
│  │  DynamoDB          │  │  Amazon S3         │                │
│  │  - User data       │  │  - Certificates    │                │
│  │  - Resource data   │  │  - Reports         │                │
│  │  - Certifications  │  │  - Badges          │                │
│  └────────────────────┘  └────────────────────┘                │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   Supporting Services                            │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐           │
│  │ AWS Cognito  │ │   AWS SNS    │ │ CloudWatch   │           │
│  │ Auth & Users │ │ Notifications│ │ Monitoring   │           │
│  └──────────────┘ └──────────────┘ └──────────────┘           │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 Technology Stack

**Frontend:**
- React.js 18+ with TypeScript
- Chart.js / Recharts for data visualization
- Service Workers for offline support
- LocalStorage/IndexedDB for offline data
- Axios for API communication

**Backend:**
- Node.js 18+ LTS
- Express.js framework
- JWT for authentication tokens
- Joi for request validation
- Winston for logging

**Database:**
- Amazon DynamoDB (NoSQL for scalability)
- DynamoDB Streams for real-time updates

**AI/ML:**
- AWS SageMaker for model training and deployment
- Python 3.9+ for ML models
- Scikit-learn, TensorFlow for model development
- AWS Lambda for serverless inference

**Cloud Services:**
- AWS API Gateway for API management
- AWS Cognito for user authentication
- AWS S3 for file storage
- AWS SNS for notifications
- AWS CloudWatch for monitoring
- AWS Lambda for serverless functions

## 2. Database Schema Design

### 2.1 DynamoDB Tables

#### Table: Users
**Purpose:** Store user account information and profiles

| Attribute | Type | Description |
|-----------|------|-------------|
| userId (PK) | String | Unique user identifier (UUID) |
| email | String | User email address |
| name | String | User full name |
| phoneNumber | String | Contact number |
| role | String | User role (USER, ADMIN, PUBLIC) |
| farmBusinessType | String | Type of farm/business |
| cropCategory | String | Primary crop category |
| productionScale | String | Scale of production (SMALL, MEDIUM, LARGE) |
| location | Map | Geographic location {latitude, longitude, address, state, district} |
| seasonalCycle | String | Seasonal cycle information |
| createdAt | Number | Timestamp of account creation |
| updatedAt | Number | Timestamp of last update |
| isActive | Boolean | Account active status |

**GSI:** email-index (for login lookup)
**GSI:** location-index (for regional queries)

#### Table: ResourceData
**Purpose:** Store waste, water, energy tracking data

| Attribute | Type | Description |
|-----------|------|-------------|
| recordId (PK) | String | Unique record identifier (UUID) |
| userId (SK) | String | User identifier |
| resourceType | String | Type (WASTE, WATER, ENERGY) |
| subType | String | Specific category (e.g., ORGANIC_WASTE, IRRIGATION) |
| quantity | Number | Measured quantity |
| unit | String | Measurement unit (KG, LITERS, KWH) |
| timestamp | Number | Recording timestamp |
| metadata | Map | Additional context data |
| createdAt | Number | Record creation timestamp |

**GSI:** userId-timestamp-index (for user timeline queries)
**GSI:** userId-resourceType-index (for filtered queries)

#### Table: SustainabilityScores
**Purpose:** Store daily sustainability scores and metrics

| Attribute | Type | Description |
|-----------|------|-------------|
| scoreId (PK) | String | Composite key: userId#date |
| userId | String | User identifier |
| date | String | Date (YYYY-MM-DD) |
| overallScore | Number | Overall sustainability score (0-100) |
| wasteScore | Number | Waste management score |
| waterScore | Number | Water efficiency score |
| energyScore | Number | Energy efficiency score |
| carbonScore | Number | Carbon footprint score |
| calculatedAt | Number | Calculation timestamp |

**GSI:** userId-date-index (for historical score queries)

#### Table: Certifications
**Purpose:** Store certification levels and history

| Attribute | Type | Description |
|-----------|------|-------------|
| certificationId (PK) | String | Unique certification identifier |
| userId | String | User identifier |
| level | String | Certification level (BRONZE, SILVER, GOLD, PLATINUM, DIAMOND) |
| status | String | Status (PENDING, APPROVED, REJECTED, REVOKED) |
| score | Number | Sustainability score at certification |
| requestedAt | Number | Request timestamp |
| approvedAt | Number | Approval timestamp |
| approvedBy | String | Admin userId who approved |
| expiresAt | Number | Expiration timestamp |
| certificateUrl | String | S3 URL for certificate PDF |
| qrCode | String | QR code data |
| rejectionReason | String | Reason if rejected |

**GSI:** userId-level-index (for user certification history)
**GSI:** status-index (for admin pending requests)

#### Table: Anomalies
**Purpose:** Store detected anomalies in resource usage

| Attribute | Type | Description |
|-----------|------|-------------|
| anomalyId (PK) | String | Unique anomaly identifier |
| userId | String | User identifier |
| resourceType | String | Type of resource (WATER, ENERGY, WASTE) |
| detectedValue | Number | Anomalous value detected |
| expectedRange | Map | {min, max, average} |
| severity | String | Severity level (LOW, MEDIUM, HIGH) |
| detectedAt | Number | Detection timestamp |
| acknowledgedAt | Number | User acknowledgment timestamp |
| isResolved | Boolean | Resolution status |

**GSI:** userId-detectedAt-index (for user anomaly history)

#### Table: Recommendations
**Purpose:** Store AI-generated recommendations

| Attribute | Type | Description |
|-----------|------|-------------|
| recommendationId (PK) | String | Unique recommendation identifier |
| userId | String | User identifier |
| type | String | Recommendation type (WASTE_TO_RESOURCE, WATER_OPTIMIZATION, ENERGY_EFFICIENCY) |
| title | String | Recommendation title |
| description | String | Detailed description |
| economicValue | Number | Estimated economic benefit |
| environmentalImpact | Number | Environmental impact score |
| implementationSteps | List | Step-by-step guidance |
| priority | Number | Priority ranking |
| generatedAt | Number | Generation timestamp |
| status | String | Status (NEW, IN_PROGRESS, COMPLETED, DISMISSED) |

**GSI:** userId-status-index (for active recommendations)

#### Table: ComplianceStatus
**Purpose:** Store compliance evaluation results

| Attribute | Type | Description |
|-----------|------|-------------|
| complianceId (PK) | String | Composite key: userId#standard |
| userId | String | User identifier |
| standard | String | Standard name (CPCB, MOEFCC, SWACHH_BHARAT, UN_SDG, ISO_14001, EPA) |
| isCompliant | Boolean | Compliance status |
| score | Number | Compliance score (0-100) |
| gaps | List | List of compliance gaps |
| remediationSteps | List | Recommended remediation actions |
| evaluatedAt | Number | Evaluation timestamp |
| nextEvaluationAt | Number | Next scheduled evaluation |

**GSI:** userId-index (for user compliance overview)

#### Table: Notifications
**Purpose:** Store notification history

| Attribute | Type | Description |
|-----------|------|-------------|
| notificationId (PK) | String | Unique notification identifier |
| userId | String | User identifier |
| type | String | Notification type (ANOMALY, CERTIFICATION, RECOMMENDATION, ALERT) |
| title | String | Notification title |
| message | String | Notification message |
| channels | List | Delivery channels (EMAIL, SMS, IN_APP) |
| sentAt | Number | Send timestamp |
| readAt | Number | Read timestamp |
| isRead | Boolean | Read status |

**GSI:** userId-sentAt-index (for user notification history)

#### Table: AuditLogs
**Purpose:** Store audit trail for sensitive operations

| Attribute | Type | Description |
|-----------|------|-------------|
| logId (PK) | String | Unique log identifier |
| userId | String | User who performed action |
| action | String | Action performed |
| resourceType | String | Type of resource affected |
| resourceId | String | Identifier of affected resource |
| timestamp | Number | Action timestamp |
| ipAddress | String | IP address of request |
| metadata | Map | Additional context data |

**GSI:** userId-timestamp-index (for user activity tracking)
**GSI:** action-timestamp-index (for action-based queries)

## 3. API Design

### 3.1 API Structure

Base URL: `https://api.agrospm.com/v1`

All API requests require authentication via JWT token in Authorization header:
```
Authorization: Bearer <jwt_token>
```

### 3.2 Authentication Endpoints

#### POST /auth/register
Register a new user account

**Request Body:**
```json
{
  "email": "farmer@example.com",
  "password": "SecurePass123!",
  "name": "John Farmer",
  "phoneNumber": "+91-9876543210",
  "farmBusinessType": "DAIRY_FARM",
  "cropCategory": "CEREALS",
  "productionScale": "MEDIUM",
  "location": {
    "latitude": 28.7041,
    "longitude": 77.1025,
    "address": "Village Name, District",
    "state": "Uttar Pradesh",
    "district": "Meerut"
  },
  "seasonalCycle": "KHARIF"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "userId": "uuid-here",
    "email": "farmer@example.com",
    "message": "Registration successful. Please check your email for confirmation."
  }
}
```

#### POST /auth/login
Authenticate user and receive JWT token

**Request Body:**
```json
{
  "email": "farmer@example.com",
  "password": "SecurePass123!"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "token": "jwt-token-here",
    "refreshToken": "refresh-token-here",
    "user": {
      "userId": "uuid-here",
      "email": "farmer@example.com",
      "name": "John Farmer",
      "role": "USER"
    }
  }
}
```

#### POST /auth/refresh
Refresh JWT token

**Request Body:**
```json
{
  "refreshToken": "refresh-token-here"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "token": "new-jwt-token-here"
  }
}
```

### 3.3 Profile Management Endpoints

#### GET /profile
Get current user profile

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "userId": "uuid-here",
    "email": "farmer@example.com",
    "name": "John Farmer",
    "phoneNumber": "+91-9876543210",
    "farmBusinessType": "DAIRY_FARM",
    "cropCategory": "CEREALS",
    "productionScale": "MEDIUM",
    "location": {
      "latitude": 28.7041,
      "longitude": 77.1025,
      "address": "Village Name, District",
      "state": "Uttar Pradesh",
      "district": "Meerut"
    },
    "seasonalCycle": "KHARIF",
    "createdAt": 1640000000000
  }
}
```

#### PUT /profile
Update user profile

**Request Body:**
```json
{
  "name": "John Updated Farmer",
  "phoneNumber": "+91-9876543211",
  "cropCategory": "VEGETABLES"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "message": "Profile updated successfully"
  }
}
```

### 3.4 Resource Tracking Endpoints

#### POST /resources/waste
Record waste data

**Request Body:**
```json
{
  "subType": "ORGANIC_WASTE",
  "quantity": 150,
  "unit": "KG",
  "timestamp": 1640000000000,
  "metadata": {
    "source": "crop_residue",
    "notes": "Post-harvest waste"
  }
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "recordId": "uuid-here",
    "message": "Waste data recorded successfully"
  }
}
```

#### POST /resources/water
Record water usage data

**Request Body:**
```json
{
  "subType": "IRRIGATION",
  "quantity": 5000,
  "unit": "LITERS",
  "timestamp": 1640000000000,
  "metadata": {
    "cropArea": "2 acres",
    "method": "drip_irrigation"
  }
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "recordId": "uuid-here",
    "message": "Water usage recorded successfully"
  }
}
```

#### POST /resources/energy
Record energy usage data

**Request Body:**
```json
{
  "subType": "MACHINERY",
  "quantity": 45,
  "unit": "KWH",
  "timestamp": 1640000000000,
  "metadata": {
    "equipment": "tractor",
    "hours": 3
  }
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "recordId": "uuid-here",
    "message": "Energy usage recorded successfully"
  }
}
```

#### GET /resources/history
Get resource usage history

**Query Parameters:**
- `resourceType`: WASTE | WATER | ENERGY (optional)
- `startDate`: ISO date string (required)
- `endDate`: ISO date string (required)
- `aggregation`: DAILY | WEEKLY | MONTHLY (optional, default: DAILY)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "records": [
      {
        "date": "2024-01-15",
        "waste": 150,
        "water": 5000,
        "energy": 45,
        "carbonFootprint": 12.5
      }
    ],
    "summary": {
      "totalWaste": 4500,
      "totalWater": 150000,
      "totalEnergy": 1350,
      "totalCarbon": 375
    }
  }
}
```

### 3.5 Dashboard Endpoints

#### GET /dashboard
Get dashboard data with all metrics

**Query Parameters:**
- `period`: DAILY | WEEKLY | MONTHLY (default: WEEKLY)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "currentMetrics": {
      "waste": 150,
      "water": 5000,
      "energy": 45,
      "carbonFootprint": 12.5
    },
    "trends": [
      {
        "date": "2024-01-15",
        "waste": 150,
        "water": 5000,
        "energy": 45,
        "carbon": 12.5
      }
    ],
    "comparisons": {
      "wasteChange": -5.2,
      "waterChange": 3.1,
      "energyChange": -2.8,
      "carbonChange": -4.5
    },
    "sustainabilityScore": 72,
    "certificationLevel": "SILVER"
  }
}
```

### 3.6 AI/ML Endpoints

#### GET /ai/predictions
Get predictive insights for resource usage

**Query Parameters:**
- `resourceType`: WATER | ENERGY (required)
- `horizon`: Number of days to predict (default: 7)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "predictions": [
      {
        "date": "2024-01-20",
        "predicted": 5200,
        "confidenceInterval": {
          "lower": 4800,
          "upper": 5600
        }
      }
    ],
    "alerts": [
      {
        "type": "SHORTAGE_WARNING",
        "message": "Water usage predicted to exceed capacity on 2024-01-22"
      }
    ]
  }
}
```

#### GET /ai/recommendations
Get AI-generated recommendations

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "recommendations": [
      {
        "recommendationId": "uuid-here",
        "type": "WASTE_TO_RESOURCE",
        "title": "Convert organic waste to biogas",
        "description": "Your organic waste volume is suitable for biogas generation",
        "economicValue": 5000,
        "environmentalImpact": 85,
        "implementationSteps": [
          "Install biogas digester",
          "Set up collection system",
          "Monitor gas production"
        ],
        "priority": 1
      }
    ]
  }
}
```

#### GET /ai/anomalies
Get detected anomalies

**Query Parameters:**
- `status`: UNRESOLVED | RESOLVED | ALL (default: UNRESOLVED)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "anomalies": [
      {
        "anomalyId": "uuid-here",
        "resourceType": "WATER",
        "detectedValue": 8500,
        "expectedRange": {
          "min": 4000,
          "max": 6000,
          "average": 5000
        },
        "severity": "HIGH",
        "detectedAt": 1640000000000,
        "isResolved": false
      }
    ]
  }
}
```

### 3.7 Sustainability Score Endpoints

#### GET /sustainability/score
Get current sustainability score

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "overallScore": 72,
    "breakdown": {
      "wasteScore": 75,
      "waterScore": 68,
      "energyScore": 70,
      "carbonScore": 74
    },
    "trend": "IMPROVING",
    "history": [
      {
        "date": "2024-01-15",
        "score": 72
      }
    ]
  }
}
```

#### GET /sustainability/roadmap
Get personalized improvement roadmap

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "currentLevel": "SILVER",
    "nextLevel": "GOLD",
    "requiredScore": 80,
    "currentScore": 72,
    "gap": 8,
    "recommendations": [
      {
        "action": "Reduce water usage by 15%",
        "impact": 3,
        "timeline": "2 months",
        "status": "NOT_STARTED"
      }
    ]
  }
}
```

### 3.8 Certification Endpoints

#### GET /certifications/current
Get current certification status

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "certificationId": "uuid-here",
    "level": "SILVER",
    "status": "APPROVED",
    "score": 72,
    "approvedAt": 1640000000000,
    "expiresAt": 1671536000000,
    "certificateUrl": "https://s3.amazonaws.com/...",
    "qrCode": "data:image/png;base64,..."
  }
}
```

#### POST /certifications/request
Request certification upgrade

**Request Body:**
```json
{
  "requestedLevel": "GOLD"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "data": {
    "certificationId": "uuid-here",
    "status": "PENDING",
    "message": "Certification request submitted for admin review"
  }
}
```

#### GET /certifications/certificate/:certificationId
Download certificate PDF

**Response (200 OK):**
Returns PDF file

#### GET /certifications/verify/:qrCode
Verify certification via QR code (public endpoint)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "isValid": true,
    "farmName": "John Farmer",
    "certificationLevel": "SILVER",
    "sustainabilityScore": 72,
    "approvedAt": 1640000000000,
    "expiresAt": 1671536000000
  }
}
```

### 3.9 Compliance Endpoints

#### GET /compliance/status
Get compliance status for all standards

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "standards": [
      {
        "standard": "CPCB",
        "isCompliant": true,
        "score": 85,
        "evaluatedAt": 1640000000000
      },
      {
        "standard": "UN_SDG",
        "isCompliant": false,
        "score": 65,
        "gaps": ["SDG 12: Responsible Consumption"],
        "remediationSteps": ["Reduce waste by 20%"],
        "evaluatedAt": 1640000000000
      }
    ]
  }
}
```

### 3.10 Public Directory Endpoints

#### GET /directory/search
Search certified producers (public endpoint)

**Query Parameters:**
- `location`: State or district name
- `cropType`: Crop category
- `certificationLevel`: Certification level
- `businessType`: Business type

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "producers": [
      {
        "userId": "uuid-here",
        "name": "John Farmer",
        "businessType": "DAIRY_FARM",
        "cropCategory": "CEREALS",
        "location": "Meerut, Uttar Pradesh",
        "certificationLevel": "SILVER",
        "sustainabilityScore": 72
      }
    ],
    "total": 1
  }
}
```

### 3.11 Reports Endpoints

#### POST /reports/impact
Generate sustainability impact report

**Request Body:**
```json
{
  "startDate": "2024-01-01",
  "endDate": "2024-01-31"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "reportUrl": "https://s3.amazonaws.com/reports/...",
    "reportId": "uuid-here"
  }
}
```

#### GET /reports/templates/social
Get social media marketing templates

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "templates": [
      {
        "platform": "FACEBOOK",
        "templateUrl": "https://s3.amazonaws.com/templates/...",
        "message": "Proud to be a Silver certified sustainable farmer!"
      }
    ]
  }
}
```

### 3.12 Admin Endpoints

#### GET /admin/users
Get all registered users (admin only)

**Query Parameters:**
- `location`: Filter by location
- `certificationLevel`: Filter by certification
- `page`: Page number
- `limit`: Items per page

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "users": [
      {
        "userId": "uuid-here",
        "name": "John Farmer",
        "email": "farmer@example.com",
        "location": "Meerut, UP",
        "certificationLevel": "SILVER",
        "sustainabilityScore": 72,
        "registeredAt": 1640000000000
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 150
    }
  }
}
```

#### GET /admin/certifications/pending
Get pending certification requests (admin only)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "requests": [
      {
        "certificationId": "uuid-here",
        "userId": "uuid-here",
        "userName": "John Farmer",
        "requestedLevel": "GOLD",
        "currentScore": 82,
        "requestedAt": 1640000000000,
        "supportingData": {
          "wasteScore": 85,
          "waterScore": 80,
          "energyScore": 82,
          "carbonScore": 81
        }
      }
    ]
  }
}
```

#### POST /admin/certifications/:certificationId/approve
Approve certification request (admin only)

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "message": "Certification approved successfully"
  }
}
```

#### POST /admin/certifications/:certificationId/reject
Reject certification request (admin only)

**Request Body:**
```json
{
  "reason": "Insufficient documentation provided"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "message": "Certification rejected"
  }
}
```

#### GET /admin/analytics/regional
Get regional sustainability analytics (admin only)

**Query Parameters:**
- `region`: State or district name
- `startDate`: ISO date string
- `endDate`: ISO date string

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "region": "Uttar Pradesh",
    "totalUsers": 1500,
    "averageScore": 68,
    "certificationDistribution": {
      "BRONZE": 500,
      "SILVER": 600,
      "GOLD": 300,
      "PLATINUM": 80,
      "DIAMOND": 20
    },
    "complianceRates": {
      "CPCB": 85,
      "UN_SDG": 72
    }
  }
}
```

#### POST /admin/reports/compliance
Generate compliance report (admin only)

**Request Body:**
```json
{
  "region": "Uttar Pradesh",
  "startDate": "2024-01-01",
  "endDate": "2024-01-31"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "reportUrl": "https://s3.amazonaws.com/admin-reports/...",
    "reportId": "uuid-here"
  }
}
```

### 3.13 Notification Endpoints

#### GET /notifications
Get user notifications

**Query Parameters:**
- `status`: READ | UNREAD | ALL (default: ALL)
- `page`: Page number
- `limit`: Items per page

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "notifications": [
      {
        "notificationId": "uuid-here",
        "type": "ANOMALY",
        "title": "High water usage detected",
        "message": "Your water usage on 2024-01-15 was 50% higher than average",
        "sentAt": 1640000000000,
        "isRead": false
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 45
    }
  }
}
```

#### PUT /notifications/:notificationId/read
Mark notification as read

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "message": "Notification marked as read"
  }
}
```

#### GET /notifications/preferences
Get notification preferences

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "email": true,
    "sms": false,
    "inApp": true,
    "types": {
      "ANOMALY": true,
      "CERTIFICATION": true,
      "RECOMMENDATION": false
    }
  }
}
```

#### PUT /notifications/preferences
Update notification preferences

**Request Body:**
```json
{
  "email": true,
  "sms": true,
  "inApp": true,
  "types": {
    "ANOMALY": true,
    "CERTIFICATION": true,
    "RECOMMENDATION": true
  }
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "message": "Notification preferences updated"
  }
}
```

## 4. AI/ML System Design

### 4.1 AI Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Data Collection Layer                     │
│  - Historical resource usage data                           │
│  - User profile data (farm type, crop, location)            │
│  - Seasonal patterns                                         │
│  - External factors (weather, market prices)                │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                  Data Preprocessing Pipeline                 │
│  - Data cleaning and validation                             │
│  - Feature engineering                                       │
│  - Normalization and scaling                                │
│  - Time series transformation                               │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                      ML Model Layer                          │
│  ┌─────────────────┐  ┌─────────────────┐                  │
│  │ Prediction      │  │ Recommendation  │                  │
│  │ Models          │  │ Engine          │                  │
│  │ - Water         │  │ - Rule-based    │                  │
│  │ - Energy        │  │ - ML-enhanced   │                  │
│  └─────────────────┘  └─────────────────┘                  │
│  ┌─────────────────┐  ┌─────────────────┐                  │
│  │ Anomaly         │  │ Scoring         │                  │
│  │ Detection       │  │ Engine          │                  │
│  │ - Statistical   │  │ - Multi-metric  │                  │
│  │ - ML-based      │  │ - Weighted      │                  │
│  └─────────────────┘  └─────────────────┘                  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│                    Inference & Serving                       │
│  - AWS Lambda for real-time inference                       │
│  - SageMaker endpoints for batch processing                 │
│  - Caching layer for frequent predictions                   │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 Predictive Models

#### 4.2.1 Water Consumption Prediction Model

**Algorithm:** Time Series Forecasting (LSTM or Prophet)

**Features:**
- Historical water usage (7-30 days)
- Crop type and growth stage
- Farm size and irrigation method
- Seasonal patterns
- Weather data (temperature, rainfall)
- Day of week

**Training Process:**
1. Collect historical water usage data from all users
2. Engineer features from raw data
3. Split data into train/validation/test sets (70/15/15)
4. Train LSTM model with hyperparameter tuning
5. Validate model performance (RMSE, MAE, MAPE)
6. Deploy model to SageMaker endpoint

**Inference:**
- Input: User profile + recent usage history
- Output: Predicted water usage for next 7 days with confidence intervals
- Latency target: < 2 seconds

#### 4.2.2 Energy Consumption Prediction Model

**Algorithm:** Time Series Forecasting (LSTM or XGBoost)

**Features:**
- Historical energy usage (7-30 days)
- Equipment type and age
- Production scale
- Seasonal patterns
- Weather data (temperature for cooling/heating)
- Day of week

**Training Process:**
Similar to water prediction model

**Inference:**
- Input: User profile + recent usage history
- Output: Predicted energy usage for next 7 days with confidence intervals
- Latency target: < 2 seconds

### 4.3 Anomaly Detection System

#### 4.3.1 Statistical Anomaly Detection

**Algorithm:** Z-score and IQR methods

**Process:**
1. Calculate rolling statistics (mean, std dev) for each resource type
2. Account for seasonal variations using seasonal decomposition
3. Detect anomalies when value exceeds threshold:
   - Z-score > 3 (HIGH severity)
   - Z-score > 2 (MEDIUM severity)
   - Z-score > 1.5 (LOW severity)
4. Generate alert with contextual information

**Implementation:**
```python
def detect_anomaly(current_value, historical_data, seasonal_factor):
    mean = np.mean(historical_data)
    std = np.std(historical_data)
    adjusted_mean = mean * seasonal_factor
    z_score = (current_value - adjusted_mean) / std
    
    if z_score > 3:
        return {"severity": "HIGH", "z_score": z_score}
    elif z_score > 2:
        return {"severity": "MEDIUM", "z_score": z_score}
    elif z_score > 1.5:
        return {"severity": "LOW", "z_score": z_score}
    return None
```

#### 4.3.2 ML-Based Anomaly Detection

**Algorithm:** Isolation Forest or Autoencoder

**Features:**
- Current resource usage
- Time of day/week/month
- Historical patterns
- Correlation with other metrics

**Training:**
- Train on normal usage patterns
- Detect outliers as anomalies
- Continuously retrain with new data

### 4.4 Waste-to-Resource Recommendation Engine

#### 4.4.1 Rule-Based Recommendations

**Organic Waste Rules:**
```python
def recommend_organic_waste(quantity, waste_type):
    recommendations = []
    
    if quantity > 100:  # kg per day
        recommendations.append({
            "type": "BIOGAS_GENERATION",
            "title": "Install biogas digester",
            "economicValue": calculate_biogas_value(quantity),
            "environmentalImpact": 90,
            "priority": 1
        })
    
    if waste_type in ["crop_residue", "manure"]:
        recommendations.append({
            "type": "COMPOSTING",
            "title": "Create compost for soil enrichment",
            "economicValue": calculate_compost_value(quantity),
            "environmentalImpact": 85,
            "priority": 2
        })
    
    return recommendations
```

**Recyclable Waste Rules:**
```python
def recommend_recyclable_waste(quantity, waste_type):
    recommendations = []
    
    if waste_type == "plastic":
        recommendations.append({
            "type": "RECYCLING",
            "title": "Sell to plastic recycling facility",
            "economicValue": quantity * 15,  # Rs per kg
            "environmentalImpact": 80,
            "buyers": find_nearby_buyers("plastic"),
            "priority": 1
        })
    
    return recommendations
```

**Industrial Agro-Waste Rules:**
```python
def recommend_industrial_waste(quantity, waste_type):
    recommendations = []
    
    if waste_type == "processing_residue":
        recommendations.append({
            "type": "ANIMAL_FEED",
            "title": "Convert to animal feed supplement",
            "economicValue": quantity * 8,  # Rs per kg
            "environmentalImpact": 75,
            "priority": 1
        })
    
    return recommendations
```

### 4.5 Sustainability Scoring Algorithm

#### 4.5.1 Score Calculation Formula

```python
def calculate_sustainability_score(user_data):
    # Weight distribution
    WEIGHTS = {
        "waste": 0.30,
        "water": 0.25,
        "energy": 0.25,
        "carbon": 0.20
    }
    
    # Calculate individual scores (0-100)
    waste_score = calculate_waste_score(user_data)
    water_score = calculate_water_score(user_data)
    energy_score = calculate_energy_score(user_data)
    carbon_score = calculate_carbon_score(user_data)
    
    # Weighted average
    overall_score = (
        waste_score * WEIGHTS["waste"] +
        water_score * WEIGHTS["water"] +
        energy_score * WEIGHTS["energy"] +
        carbon_score * WEIGHTS["carbon"]
    )
    
    return {
        "overallScore": round(overall_score, 2),
        "breakdown": {
            "wasteScore": waste_score,
            "waterScore": water_score,
            "energyScore": energy_score,
            "carbonScore": carbon_score
        }
    }
```

#### 4.5.2 Individual Metric Scoring

**Waste Score:**
```python
def calculate_waste_score(user_data):
    # Factors:
    # - Waste reduction over time
    # - Recycling rate
    # - Waste-to-resource conversion rate
    
    baseline_waste = user_data["baseline_waste"]
    current_waste = user_data["current_waste"]
    recycling_rate = user_data["recycling_rate"]
    conversion_rate = user_data["conversion_rate"]
    
    reduction_score = min(100, ((baseline_waste - current_waste) / baseline_waste) * 100)
    recycling_score = recycling_rate * 100
    conversion_score = conversion_rate * 100
    
    waste_score = (
        reduction_score * 0.4 +
        recycling_score * 0.3 +
        conversion_score * 0.3
    )
    
    return max(0, min(100, waste_score))
```

**Water Score:**
```python
def calculate_water_score(user_data):
    # Factors:
    # - Water efficiency (output per liter)
    # - Reduction over time
    # - Conservation practices
    
    efficiency = user_data["water_efficiency"]
    reduction = user_data["water_reduction"]
    conservation_practices = user_data["conservation_practices"]
    
    efficiency_score = min(100, efficiency * 50)
    reduction_score = min(100, reduction * 100)
    practices_score = len(conservation_practices) * 20
    
    water_score = (
        efficiency_score * 0.4 +
        reduction_score * 0.4 +
        practices_score * 0.2
    )
    
    return max(0, min(100, water_score))
```

**Energy Score:**
```python
def calculate_energy_score(user_data):
    # Factors:
    # - Energy efficiency (output per kWh)
    # - Renewable energy usage
    # - Reduction over time
    
    efficiency = user_data["energy_efficiency"]
    renewable_percentage = user_data["renewable_percentage"]
    reduction = user_data["energy_reduction"]
    
    efficiency_score = min(100, efficiency * 50)
    renewable_score = renewable_percentage * 100
    reduction_score = min(100, reduction * 100)
    
    energy_score = (
        efficiency_score * 0.4 +
        renewable_score * 0.3 +
        reduction_score * 0.3
    )
    
    return max(0, min(100, energy_score))
```

**Carbon Score:**
```python
def calculate_carbon_score(user_data):
    # Factors:
    # - Carbon footprint per unit output
    # - Reduction over time
    # - Carbon offset activities
    
    footprint_per_unit = user_data["carbon_per_unit"]
    reduction = user_data["carbon_reduction"]
    offset_activities = user_data["offset_activities"]
    
    footprint_score = max(0, 100 - (footprint_per_unit * 10))
    reduction_score = min(100, reduction * 100)
    offset_score = len(offset_activities) * 15
    
    carbon_score = (
        footprint_score * 0.5 +
        reduction_score * 0.3 +
        offset_score * 0.2
    )
    
    return max(0, min(100, carbon_score))
```

### 4.6 Personalized Improvement Roadmap

**Algorithm:**
```python
def generate_improvement_roadmap(user_data, target_level):
    current_score = user_data["sustainability_score"]
    target_score = CERTIFICATION_THRESHOLDS[target_level]
    gap = target_score - current_score
    
    # Identify weakest metrics
    breakdown = user_data["score_breakdown"]
    weak_metrics = sorted(breakdown.items(), key=lambda x: x[1])
    
    recommendations = []
    
    for metric, score in weak_metrics:
        if gap <= 0:
            break
        
        # Generate targeted recommendations
        metric_recommendations = generate_metric_recommendations(
            metric, score, user_data
        )
        
        for rec in metric_recommendations:
            recommendations.append({
                "metric": metric,
                "action": rec["action"],
                "impact": rec["impact"],
                "timeline": rec["timeline"],
                "difficulty": rec["difficulty"],
                "priority": calculate_priority(rec, gap)
            })
            gap -= rec["impact"]
    
    return sorted(recommendations, key=lambda x: x["priority"], reverse=True)
```

## 5. Certification System Design

### 5.1 Certification Level Thresholds

```python
CERTIFICATION_THRESHOLDS = {
    "BRONZE": 40,
    "SILVER": 60,
    "GOLD": 75,
    "PLATINUM": 85,
    "DIAMOND": 95
}

ADMIN_VERIFICATION_REQUIRED = ["GOLD", "PLATINUM", "DIAMOND"]
```

### 5.2 Certification Workflow

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Score Calculation                                  │
│  - Daily sustainability score calculated                    │
│  - 30-day rolling average computed                          │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 2: Eligibility Check                                  │
│  - Check if score meets threshold                           │
│  - Verify score stability (maintained for 7+ days)          │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 3: Certification Request                              │
│  - Auto-request for Bronze/Silver                           │
│  - Manual request for Gold/Platinum/Diamond                 │
└─────────────────────────────────────────────────────────────┘
                            ↓
                    ┌───────┴───────┐
                    │               │
        ┌───────────▼─────┐   ┌────▼──────────┐
        │ Bronze/Silver   │   │ Gold/Platinum │
        │ Auto-Approve    │   │ Admin Review  │
        └───────────┬─────┘   └────┬──────────┘
                    │               │
                    └───────┬───────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 4: Certificate Generation                             │
│  - Generate PDF certificate                                 │
│  - Create QR code                                           │
│  - Generate digital badges                                  │
│  - Store in S3                                              │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 5: Notification & Directory Update                    │
│  - Notify user via email/SMS/in-app                         │
│  - Add to public directory                                  │
│  - Generate social media templates                          │
└─────────────────────────────────────────────────────────────┘
```

### 5.3 Certificate Template Design

**Certificate Components:**
- AgroSPM logo and branding
- Certification level badge (color-coded)
- User/farm name
- Certification level text
- Sustainability score
- Issue date and expiration date
- Unique certificate ID
- QR code for verification
- Digital signature
- Compliance standards met

**Badge Colors:**
- Bronze: #CD7F32
- Silver: #C0C0C0
- Gold: #FFD700
- Platinum: #E5E4E2
- Diamond: #B9F2FF

### 5.4 QR Code Verification System

**QR Code Data Structure:**
```json
{
  "version": "1.0",
  "certificationId": "uuid-here",
  "verificationUrl": "https://agrospm.com/verify/uuid-here"
}
```

**Verification Flow:**
1. User scans QR code
2. Mobile device opens verification URL
3. Public API endpoint validates certification
4. Display public profile with:
   - Farm/business name
   - Certification level
   - Sustainability score
   - Approval date
   - Expiration date
   - Verification status

**Security:**
- QR codes are cryptographically signed
- Verification URL includes tamper-proof token
- Expired certifications show "EXPIRED" status
- Revoked certifications show "INVALID" status

## 6. Compliance Evaluation System

### 6.1 Compliance Standards Mapping

#### 6.1.1 CPCB (Central Pollution Control Board)

**Evaluation Criteria:**
```python
def evaluate_cpcb_compliance(user_data):
    criteria = {
        "waste_disposal": check_waste_disposal_methods(user_data),
        "water_pollution": check_water_pollution_levels(user_data),
        "air_quality": check_air_quality_impact(user_data),
        "hazardous_waste": check_hazardous_waste_handling(user_data)
    }
    
    compliance_score = sum(criteria.values()) / len(criteria) * 100
    is_compliant = compliance_score >= 70
    
    gaps = [k for k, v in criteria.items() if not v]
    
    return {
        "standard": "CPCB",
        "isCompliant": is_compliant,
        "score": compliance_score,
        "gaps": gaps,
        "remediationSteps": generate_remediation_steps(gaps)
    }
```

#### 6.1.2 MoEFCC Waste Management Rules

**Evaluation Criteria:**
```python
def evaluate_moefcc_compliance(user_data):
    criteria = {
        "segregation": check_waste_segregation(user_data),
        "storage": check_waste_storage_practices(user_data),
        "disposal": check_disposal_compliance(user_data),
        "recycling": check_recycling_rate(user_data)
    }
    
    compliance_score = sum(criteria.values()) / len(criteria) * 100
    is_compliant = compliance_score >= 75
    
    return {
        "standard": "MOEFCC",
        "isCompliant": is_compliant,
        "score": compliance_score
    }
```

#### 6.1.3 UN Sustainable Development Goals (SDGs)

**Relevant SDGs:**
- SDG 2: Zero Hunger (sustainable agriculture)
- SDG 6: Clean Water and Sanitation
- SDG 7: Affordable and Clean Energy
- SDG 12: Responsible Consumption and Production
- SDG 13: Climate Action
- SDG 15: Life on Land

**Evaluation:**
```python
def evaluate_un_sdg_compliance(user_data):
    sdg_scores = {
        "SDG_2": evaluate_sustainable_agriculture(user_data),
        "SDG_6": evaluate_water_management(user_data),
        "SDG_7": evaluate_clean_energy(user_data),
        "SDG_12": evaluate_responsible_consumption(user_data),
        "SDG_13": evaluate_climate_action(user_data),
        "SDG_15": evaluate_land_management(user_data)
    }
    
    overall_score = sum(sdg_scores.values()) / len(sdg_scores)
    is_compliant = overall_score >= 65
    
    return {
        "standard": "UN_SDG",
        "isCompliant": is_compliant,
        "score": overall_score,
        "breakdown": sdg_scores
    }
```

#### 6.1.4 ISO 14001 Environmental Management

**Evaluation Criteria:**
```python
def evaluate_iso_14001_compliance(user_data):
    criteria = {
        "environmental_policy": check_policy_existence(user_data),
        "planning": check_environmental_planning(user_data),
        "implementation": check_implementation_practices(user_data),
        "monitoring": check_monitoring_systems(user_data),
        "improvement": check_continuous_improvement(user_data)
    }
    
    compliance_score = sum(criteria.values()) / len(criteria) * 100
    is_compliant = compliance_score >= 80
    
    return {
        "standard": "ISO_14001",
        "isCompliant": is_compliant,
        "score": compliance_score
    }
```

#### 6.1.5 EPA Sustainability Guidelines

**Evaluation Criteria:**
```python
def evaluate_epa_compliance(user_data):
    criteria = {
        "pollution_prevention": check_pollution_prevention(user_data),
        "resource_conservation": check_resource_conservation(user_data),
        "waste_minimization": check_waste_minimization(user_data),
        "environmental_stewardship": check_stewardship(user_data)
    }
    
    compliance_score = sum(criteria.values()) / len(criteria) * 100
    is_compliant = compliance_score >= 70
    
    return {
        "standard": "EPA",
        "isCompliant": is_compliant,
        "score": compliance_score
    }
```

## 7. Offline-First Architecture

### 7.1 Offline Data Storage

**LocalStorage Structure:**
```javascript
{
  "user": {
    "userId": "uuid-here",
    "profile": {...}
  },
  "pendingSync": [
    {
      "id": "local-uuid",
      "type": "RESOURCE_DATA",
      "action": "CREATE",
      "data": {...},
      "timestamp": 1640000000000
    }
  ],
  "cachedData": {
    "dashboard": {...},
    "scores": {...},
    "certifications": {...}
  },
  "lastSync": 1640000000000
}
```

### 7.2 Synchronization Strategy

**Sync Process:**
```javascript
async function synchronizeData() {
  // 1. Check connectivity
  if (!navigator.onLine) {
    return { status: "OFFLINE" };
  }
  
  // 2. Get pending operations
  const pending = getPendingOperations();
  
  // 3. Sort by timestamp (oldest first)
  pending.sort((a, b) => a.timestamp - b.timestamp);
  
  // 4. Sync each operation
  const results = [];
  for (const operation of pending) {
    try {
      const result = await syncOperation(operation);
      results.push({ id: operation.id, status: "SUCCESS" });
      removePendingOperation(operation.id);
    } catch (error) {
      if (error.code === "CONFLICT") {
        // Handle conflict
        const resolved = await resolveConflict(operation, error.serverData);
        results.push({ id: operation.id, status: "RESOLVED" });
      } else {
        results.push({ id: operation.id, status: "FAILED", error });
      }
    }
  }
  
  // 5. Update last sync timestamp
  setLastSync(Date.now());
  
  return { status: "COMPLETED", results };
}
```

### 7.3 Conflict Resolution

**Strategy:** Last-Write-Wins with Timestamp

```javascript
function resolveConflict(localData, serverData) {
  if (localData.timestamp > serverData.timestamp) {
    // Local data is newer, push to server
    return forceUpdateServer(localData);
  } else {
    // Server data is newer, update local
    updateLocalData(serverData);
    return { resolved: "SERVER_WINS" };
  }
}
```

## 8. Security Architecture

### 8.1 Authentication Flow

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: User Login                                         │
│  - User provides email/password                             │
│  - Frontend sends to /auth/login                            │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 2: AWS Cognito Authentication                         │
│  - Cognito validates credentials                            │
│  - Returns JWT access token and refresh token               │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 3: Token Storage                                      │
│  - Access token stored in memory (short-lived: 1 hour)      │
│  - Refresh token stored in httpOnly cookie (7 days)         │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 4: API Requests                                       │
│  - Include access token in Authorization header             │
│  - Backend validates token with Cognito                     │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 5: Token Refresh                                      │
│  - When access token expires, use refresh token             │
│  - Get new access token from /auth/refresh                  │
└─────────────────────────────────────────────────────────────┘
```

### 8.2 Authorization Model

**Role-Based Access Control (RBAC):**

```javascript
const PERMISSIONS = {
  USER: [
    "read:own_profile",
    "write:own_profile",
    "read:own_resources",
    "write:own_resources",
    "read:own_dashboard",
    "read:own_certifications",
    "write:certification_request",
    "read:public_directory"
  ],
  ADMIN: [
    "read:all_users",
    "read:all_resources",
    "write:certification_approval",
    "read:admin_analytics",
    "write:admin_reports",
    "read:audit_logs"
  ],
  PUBLIC: [
    "read:public_directory",
    "read:certification_verification"
  ]
};

function checkPermission(user, permission) {
  const userPermissions = PERMISSIONS[user.role] || [];
  return userPermissions.includes(permission);
}
```

### 8.3 Data Encryption

**In Transit:**
- TLS 1.3 for all API communications
- Certificate pinning for mobile apps
- HTTPS enforcement

**At Rest:**
- DynamoDB encryption using AWS KMS
- S3 bucket encryption (AES-256)
- Encrypted backups

**Sensitive Data:**
```javascript
// PII fields encrypted before storage
const ENCRYPTED_FIELDS = [
  "email",
  "phoneNumber",
  "location.address"
];

function encryptSensitiveData(data) {
  const encrypted = { ...data };
  ENCRYPTED_FIELDS.forEach(field => {
    if (getNestedValue(encrypted, field)) {
      setNestedValue(encrypted, field, encrypt(getNestedValue(encrypted, field)));
    }
  });
  return encrypted;
}
```

### 8.4 API Security

**Rate Limiting:**
```javascript
const RATE_LIMITS = {
  "/auth/login": { requests: 5, window: "15m" },
  "/auth/register": { requests: 3, window: "1h" },
  "/resources/*": { requests: 100, window: "1h" },
  "/ai/*": { requests: 50, window: "1h" },
  "default": { requests: 1000, window: "1h" }
};
```

**Input Validation:**
```javascript
const resourceSchema = Joi.object({
  subType: Joi.string().valid("ORGANIC_WASTE", "RECYCLABLE_WASTE", "INDUSTRIAL_WASTE").required(),
  quantity: Joi.number().positive().max(1000000).required(),
  unit: Joi.string().valid("KG", "TONS", "LITERS", "CUBIC_METERS", "KWH", "MJ").required(),
  timestamp: Joi.number().integer().positive().required(),
  metadata: Joi.object().optional()
});
```

**SQL Injection Prevention:**
- Use parameterized queries
- Input sanitization
- ORM/ODM usage (DynamoDB SDK)

**XSS Prevention:**
- Content Security Policy headers
- Input sanitization
- Output encoding

## 9. Performance Optimization

### 9.1 Caching Strategy

**Multi-Layer Caching:**

```
┌─────────────────────────────────────────────────────────────┐
│  Layer 1: Browser Cache                                     │
│  - Static assets (JS, CSS, images)                          │
│  - Cache duration: 1 year with versioning                   │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Layer 2: CDN Cache (CloudFront)                            │
│  - Static assets                                            │
│  - API responses for public endpoints                       │
│  - Cache duration: 24 hours                                 │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Layer 3: Application Cache (Redis/ElastiCache)             │
│  - Dashboard data: 5 minutes                                │
│  - Sustainability scores: 1 hour                            │
│  - User profiles: 15 minutes                                │
│  - AI predictions: 6 hours                                  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Layer 4: Database                                          │
│  - DynamoDB with DAX (DynamoDB Accelerator)                 │
│  - Microsecond latency for cached queries                   │
└─────────────────────────────────────────────────────────────┘
```

### 9.2 Database Optimization

**DynamoDB Best Practices:**

1. **Partition Key Design:**
   - Use userId as partition key for user-specific data
   - Composite keys for time-series data (userId#timestamp)
   - Avoid hot partitions

2. **Global Secondary Indexes (GSI):**
   - Create GSIs for common query patterns
   - Project only required attributes
   - Monitor GSI capacity

3. **Query Optimization:**
   ```javascript
   // Good: Query with partition key
   const params = {
     TableName: "ResourceData",
     KeyConditionExpression: "userId = :userId AND #ts BETWEEN :start AND :end",
     ExpressionAttributeNames: { "#ts": "timestamp" },
     ExpressionAttributeValues: {
       ":userId": userId,
       ":start": startTimestamp,
       ":end": endTimestamp
     }
   };
   
   // Bad: Scan entire table
   // const params = { TableName: "ResourceData" };
   ```

4. **Batch Operations:**
   ```javascript
   // Use BatchWriteItem for multiple writes
   const params = {
     RequestItems: {
       "ResourceData": items.map(item => ({
         PutRequest: { Item: item }
       }))
     }
   };
   ```

### 9.3 API Response Optimization

**Pagination:**
```javascript
function paginateResults(items, page = 1, limit = 20) {
  const startIndex = (page - 1) * limit;
  const endIndex = startIndex + limit;
  
  return {
    data: items.slice(startIndex, endIndex),
    pagination: {
      page,
      limit,
      total: items.length,
      totalPages: Math.ceil(items.length / limit)
    }
  };
}
```

**Response Compression:**
```javascript
// Enable gzip compression
app.use(compression({
  level: 6,
  threshold: 1024, // Only compress responses > 1KB
  filter: (req, res) => {
    if (req.headers['x-no-compression']) {
      return false;
    }
    return compression.filter(req, res);
  }
}));
```

### 9.4 Frontend Optimization

**Code Splitting:**
```javascript
// Lazy load routes
const Dashboard = lazy(() => import('./pages/Dashboard'));
const Analytics = lazy(() => import('./pages/Analytics'));
const Certifications = lazy(() => import('./pages/Certifications'));

// Route configuration
<Suspense fallback={<LoadingSpinner />}>
  <Routes>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/analytics" element={<Analytics />} />
    <Route path="/certifications" element={<Certifications />} />
  </Routes>
</Suspense>
```

**Image Optimization:**
- Use WebP format with fallback
- Lazy load images below the fold
- Responsive images with srcset
- Compress images before upload

**Bundle Optimization:**
- Tree shaking for unused code
- Minification and uglification
- Separate vendor bundles
- Dynamic imports for large libraries

### 9.5 Auto-Scaling Configuration

**AWS Lambda:**
```yaml
functions:
  sustainabilityScore:
    handler: handlers/score.calculate
    memorySize: 512
    timeout: 30
    reservedConcurrency: 100
    provisionedConcurrency: 10
```

**API Gateway:**
- Throttling: 10,000 requests per second
- Burst: 5,000 requests
- Per-user quota: 1,000 requests per hour

**DynamoDB:**
- On-demand capacity mode for variable workloads
- Auto-scaling for provisioned capacity
- Target utilization: 70%

## 10. Monitoring and Observability

### 10.1 CloudWatch Metrics

**Custom Metrics:**
```javascript
const metrics = {
  "SustainabilityScoreCalculation": {
    unit: "Milliseconds",
    dimensions: { Service: "AI" }
  },
  "ResourceDataIngestion": {
    unit: "Count",
    dimensions: { ResourceType: "WASTE|WATER|ENERGY" }
  },
  "CertificationRequests": {
    unit: "Count",
    dimensions: { Level: "BRONZE|SILVER|GOLD|PLATINUM|DIAMOND" }
  },
  "APILatency": {
    unit: "Milliseconds",
    dimensions: { Endpoint: "/api/path" }
  }
};
```

### 10.2 Logging Strategy

**Log Levels:**
- ERROR: System errors, exceptions
- WARN: Anomalies, degraded performance
- INFO: Important business events
- DEBUG: Detailed diagnostic information

**Structured Logging:**
```javascript
logger.info("Resource data recorded", {
  userId: "uuid-here",
  resourceType: "WATER",
  quantity: 5000,
  timestamp: Date.now(),
  requestId: "req-uuid"
});
```

### 10.3 Alerting

**Critical Alerts:**
- API error rate > 5%
- Database latency > 2 seconds
- Lambda function errors
- Authentication failures spike
- System downtime

**Warning Alerts:**
- API latency > 1 second
- Cache hit rate < 80%
- Disk usage > 80%
- Memory usage > 85%

## 11. Deployment Architecture

### 11.1 Environment Configuration

**Development:**
- Single region deployment
- Minimal resources
- Debug logging enabled
- No auto-scaling

**Staging:**
- Production-like configuration
- Reduced capacity
- Integration testing
- Performance testing

**Production:**
- Multi-region deployment (primary: Mumbai, backup: Singapore)
- Full auto-scaling
- Production logging
- High availability configuration

### 11.2 CI/CD Pipeline

```
┌─────────────────────────────────────────────────────────────┐
│  Step 1: Code Commit                                        │
│  - Developer pushes to Git repository                       │
│  - Triggers CI/CD pipeline                                  │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 2: Build                                              │
│  - Install dependencies                                     │
│  - Run linters (ESLint, Prettier)                           │
│  - Compile TypeScript                                       │
│  - Build frontend bundle                                    │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 3: Test                                               │
│  - Run unit tests (Jest)                                    │
│  - Run integration tests                                    │
│  - Run property-based tests                                 │
│  - Generate coverage report (>80% required)                 │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 4: Security Scan                                      │
│  - Dependency vulnerability scan                            │
│  - SAST (Static Application Security Testing)               │
│  - Container image scanning                                 │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 5: Deploy to Staging                                  │
│  - Deploy backend to Lambda                                 │
│  - Deploy frontend to S3 + CloudFront                       │
│  - Run smoke tests                                          │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 6: Manual Approval                                    │
│  - QA team reviews staging                                  │
│  - Approve for production deployment                        │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│  Step 7: Deploy to Production                               │
│  - Blue-green deployment                                    │
│  - Gradual traffic shift (10% → 50% → 100%)                │
│  - Monitor metrics and errors                               │
│  - Automatic rollback on errors                             │
└─────────────────────────────────────────────────────────────┘
```

### 11.3 Disaster Recovery

**Backup Strategy:**
- DynamoDB: Point-in-time recovery enabled
- S3: Versioning and cross-region replication
- Lambda: Code stored in version control
- Configuration: Infrastructure as Code (Terraform/CloudFormation)

**Recovery Time Objective (RTO):** 1 hour
**Recovery Point Objective (RPO):** 5 minutes

**Failover Process:**
1. Detect primary region failure
2. Route traffic to backup region
3. Restore data from backups if needed
4. Verify system functionality
5. Monitor for issues

## 12. Frontend Component Architecture

### 12.1 Component Hierarchy

```
App
├── AuthProvider
│   ├── LoginPage
│   ├── RegisterPage
│   └── PasswordResetPage
├── PrivateRoute
│   ├── DashboardLayout
│   │   ├── Header
│   │   │   ├── Logo
│   │   │   ├── Navigation
│   │   │   └── UserMenu
│   │   ├── Sidebar
│   │   │   ├── NavLinks
│   │   │   └── CertificationBadge
│   │   └── MainContent
│   │       ├── Dashboard
│   │       │   ├── MetricsOverview
│   │       │   ├── TrendCharts
│   │       │   ├── SustainabilityScore
│   │       │   └── RecentAlerts
│   │       ├── ResourceTracking
│   │       │   ├── WasteTracker
│   │       │   ├── WaterTracker
│   │       │   └── EnergyTracker
│   │       ├── Analytics
│   │       │   ├── TimeRangeSelector
│   │       │   ├── ResourceCharts
│   │       │   └── ComparativeAnalytics
│   │       ├── AIInsights
│   │       │   ├── Predictions
│   │       │   ├── Recommendations
│   │       │   └── Anomalies
│   │       ├── Certifications
│   │       │   ├── CurrentCertification
│   │       │   ├── CertificationHistory
│   │       │   ├── ImprovementRoadmap
│   │       │   └── DownloadCertificate
│   │       ├── Compliance
│   │       │   ├── ComplianceOverview
│   │       │   └── StandardDetails
│   │       ├── Reports
│   │       │   ├── ImpactReportGenerator
│   │       │   └── SocialMediaTemplates
│   │       └── Profile
│   │           ├── ProfileForm
│   │           └── NotificationPreferences
│   └── AdminLayout
│       ├── AdminHeader
│       ├── AdminSidebar
│       └── AdminContent
│           ├── UserManagement
│           ├── CertificationReview
│           ├── Analytics
│           └── ComplianceReports
└── PublicRoute
    ├── PublicDirectory
    │   ├── SearchFilters
    │   └── ProducerList
    └── CertificationVerification
```

### 12.2 State Management

**Context API Structure:**
```javascript
// AuthContext
const AuthContext = createContext({
  user: null,
  token: null,
  login: () => {},
  logout: () => {},
  isAuthenticated: false
});

// DashboardContext
const DashboardContext = createContext({
  metrics: null,
  trends: null,
  score: null,
  loading: false,
  error: null,
  refreshDashboard: () => {}
});

// NotificationContext
const NotificationContext = createContext({
  notifications: [],
  unreadCount: 0,
  markAsRead: () => {},
  fetchNotifications: () => {}
});
```

### 12.3 Key Components

#### MetricsOverview Component
```javascript
function MetricsOverview({ metrics }) {
  return (
    <div className="metrics-grid">
      <MetricCard
        title="Waste Generated"
        value={metrics.waste}
        unit="kg"
        trend={metrics.wasteTrend}
        icon={<WasteIcon />}
      />
      <MetricCard
        title="Water Usage"
        value={metrics.water}
        unit="liters"
        trend={metrics.waterTrend}
        icon={<WaterIcon />}
      />
      <MetricCard
        title="Energy Consumption"
        value={metrics.energy}
        unit="kWh"
        trend={metrics.energyTrend}
        icon={<EnergyIcon />}
      />
      <MetricCard
        title="Carbon Footprint"
        value={metrics.carbon}
        unit="kg CO2"
        trend={metrics.carbonTrend}
        icon={<CarbonIcon />}
      />
    </div>
  );
}
```

#### TrendCharts Component
```javascript
function TrendCharts({ data, period }) {
  const chartData = useMemo(() => 
    formatChartData(data, period), 
    [data, period]
  );
  
  return (
    <div className="charts-container">
      <LineChart
        data={chartData}
        lines={[
          { key: "waste", color: "#CD7F32", label: "Waste" },
          { key: "water", color: "#1E90FF", label: "Water" },
          { key: "energy", color: "#FFD700", label: "Energy" },
          { key: "carbon", color: "#32CD32", label: "Carbon" }
        ]}
        xAxis="date"
        yAxis="value"
      />
    </div>
  );
}
```

#### SustainabilityScore Component
```javascript
function SustainabilityScore({ score, breakdown, trend }) {
  return (
    <div className="score-card">
      <div className="score-display">
        <CircularProgress value={score} max={100} />
        <div className="score-value">{score}</div>
        <div className="score-trend">
          {trend === "IMPROVING" ? <TrendUpIcon /> : <TrendDownIcon />}
          {trend}
        </div>
      </div>
      <div className="score-breakdown">
        <BreakdownBar label="Waste" value={breakdown.wasteScore} />
        <BreakdownBar label="Water" value={breakdown.waterScore} />
        <BreakdownBar label="Energy" value={breakdown.energyScore} />
        <BreakdownBar label="Carbon" value={breakdown.carbonScore} />
      </div>
    </div>
  );
}
```

#### ResourceTracker Component
```javascript
function ResourceTracker({ resourceType }) {
  const [formData, setFormData] = useState({
    subType: "",
    quantity: "",
    unit: "",
    metadata: {}
  });
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await api.post(`/resources/${resourceType}`, {
        ...formData,
        timestamp: Date.now()
      });
      showSuccessNotification("Data recorded successfully");
      resetForm();
    } catch (error) {
      showErrorNotification(error.message);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <Select
        label="Type"
        value={formData.subType}
        onChange={(value) => setFormData({ ...formData, subType: value })}
        options={getSubTypeOptions(resourceType)}
      />
      <Input
        label="Quantity"
        type="number"
        value={formData.quantity}
        onChange={(value) => setFormData({ ...formData, quantity: value })}
      />
      <Select
        label="Unit"
        value={formData.unit}
        onChange={(value) => setFormData({ ...formData, unit: value })}
        options={getUnitOptions(resourceType)}
      />
      <Button type="submit">Record Data</Button>
    </form>
  );
}
```

## 13. Testing Strategy

### 13.1 Unit Testing

**Framework:** Jest + React Testing Library

**Coverage Requirements:**
- Overall: > 80%
- Critical paths: > 95%
- Utility functions: 100%

**Example Tests:**
```javascript
// Sustainability score calculation
describe("calculateSustainabilityScore", () => {
  test("should calculate correct overall score", () => {
    const userData = {
      wasteScore: 75,
      waterScore: 68,
      energyScore: 70,
      carbonScore: 74
    };
    
    const result = calculateSustainabilityScore(userData);
    
    expect(result.overallScore).toBeCloseTo(72, 1);
  });
  
  test("should handle edge case of zero scores", () => {
    const userData = {
      wasteScore: 0,
      waterScore: 0,
      energyScore: 0,
      carbonScore: 0
    };
    
    const result = calculateSustainabilityScore(userData);
    
    expect(result.overallScore).toBe(0);
  });
});
```

### 13.2 Integration Testing

**Framework:** Jest + Supertest

**Test Scenarios:**
- Complete user registration flow
- Resource data recording and retrieval
- Certification request and approval workflow
- Dashboard data aggregation
- AI prediction generation

**Example Test:**
```javascript
describe("Resource Tracking Integration", () => {
  let authToken;
  let userId;
  
  beforeAll(async () => {
    // Create test user and authenticate
    const response = await request(app)
      .post("/auth/register")
      .send(testUserData);
    
    userId = response.body.data.userId;
    authToken = response.body.data.token;
  });
  
  test("should record and retrieve waste data", async () => {
    // Record waste data
    const wasteData = {
      subType: "ORGANIC_WASTE",
      quantity: 150,
      unit: "KG",
      timestamp: Date.now()
    };
    
    await request(app)
      .post("/resources/waste")
      .set("Authorization", `Bearer ${authToken}`)
      .send(wasteData)
      .expect(201);
    
    // Retrieve waste data
    const response = await request(app)
      .get("/resources/history")
      .set("Authorization", `Bearer ${authToken}`)
      .query({ resourceType: "WASTE" })
      .expect(200);
    
    expect(response.body.data.records).toHaveLength(1);
    expect(response.body.data.records[0].quantity).toBe(150);
  });
});
```

### 13.3 Property-Based Testing

**Framework:** fast-check

**Test Properties:**
```javascript
import fc from "fast-check";

describe("Sustainability Score Properties", () => {
  test("score should always be between 0 and 100", () => {
    fc.assert(
      fc.property(
        fc.record({
          wasteScore: fc.integer({ min: 0, max: 100 }),
          waterScore: fc.integer({ min: 0, max: 100 }),
          energyScore: fc.integer({ min: 0, max: 100 }),
          carbonScore: fc.integer({ min: 0, max: 100 })
        }),
        (userData) => {
          const result = calculateSustainabilityScore(userData);
          return result.overallScore >= 0 && result.overallScore <= 100;
        }
      )
    );
  });
  
  test("higher individual scores should result in higher overall score", () => {
    fc.assert(
      fc.property(
        fc.record({
          wasteScore: fc.integer({ min: 0, max: 100 }),
          waterScore: fc.integer({ min: 0, max: 100 }),
          energyScore: fc.integer({ min: 0, max: 100 }),
          carbonScore: fc.integer({ min: 0, max: 100 })
        }),
        (userData1) => {
          const userData2 = {
            wasteScore: Math.min(100, userData1.wasteScore + 10),
            waterScore: Math.min(100, userData1.waterScore + 10),
            energyScore: Math.min(100, userData1.energyScore + 10),
            carbonScore: Math.min(100, userData1.carbonScore + 10)
          };
          
          const score1 = calculateSustainabilityScore(userData1);
          const score2 = calculateSustainabilityScore(userData2);
          
          return score2.overallScore >= score1.overallScore;
        }
      )
    );
  });
});
```

### 13.4 End-to-End Testing

**Framework:** Playwright or Cypress

**Test Scenarios:**
- User registration and login
- Complete resource tracking workflow
- Dashboard visualization and interaction
- Certification request and download
- Admin certification approval
- Public directory search

**Example Test:**
```javascript
describe("User Journey: Resource Tracking to Certification", () => {
  test("complete user journey", async () => {
    // 1. Register and login
    await page.goto("/register");
    await page.fill("#email", "test@example.com");
    await page.fill("#password", "SecurePass123!");
    await page.fill("#name", "Test Farmer");
    await page.click("button[type=submit]");
    
    // 2. Complete profile
    await page.goto("/profile");
    await page.selectOption("#farmType", "DAIRY_FARM");
    await page.selectOption("#cropCategory", "CEREALS");
    await page.click("button[type=submit]");
    
    // 3. Record resource data
    await page.goto("/resources/waste");
    await page.selectOption("#subType", "ORGANIC_WASTE");
    await page.fill("#quantity", "150");
    await page.selectOption("#unit", "KG");
    await page.click("button[type=submit]");
    
    // 4. View dashboard
    await page.goto("/dashboard");
    await expect(page.locator(".sustainability-score")).toBeVisible();
    
    // 5. Check certification eligibility
    await page.goto("/certifications");
    const certLevel = await page.locator(".current-certification").textContent();
    expect(certLevel).toContain("BRONZE");
  });
});
```

### 13.5 Performance Testing

**Tools:** Apache JMeter, k6

**Test Scenarios:**
- Load testing: 100,000 concurrent users
- Stress testing: Gradual load increase until failure
- Spike testing: Sudden traffic spikes
- Endurance testing: Sustained load over 24 hours

**Performance Targets:**
- API response time: < 2 seconds (95th percentile)
- Dashboard load time: < 3 seconds
- AI prediction time: < 5 seconds
- Database query time: < 500ms
- Error rate: < 0.1%

## 14. Cost Optimization

### 14.1 AWS Cost Estimates (Monthly)

**Compute:**
- Lambda: $200 (1M requests/day)
- API Gateway: $150 (1M requests/day)

**Storage:**
- DynamoDB: $300 (100GB data, on-demand)
- S3: $50 (500GB storage)

**AI/ML:**
- SageMaker: $400 (model training and inference)

**Networking:**
- CloudFront: $100 (1TB data transfer)
- Data transfer: $50

**Other Services:**
- Cognito: $50 (100K MAU)
- SNS: $20 (notifications)
- CloudWatch: $30 (logs and metrics)

**Total Estimated Cost:** ~$1,350/month for 100K users

### 14.2 Cost Optimization Strategies

1. **Use Reserved Capacity:** Save 30-50% on predictable workloads
2. **Implement Caching:** Reduce database and API calls
3. **Optimize Lambda Memory:** Right-size function memory
4. **Use S3 Lifecycle Policies:** Move old data to cheaper storage tiers
5. **Compress Data:** Reduce storage and transfer costs
6. **Monitor and Alert:** Set up cost anomaly detection

## 15. Conclusion

This design document provides a comprehensive blueprint for implementing the AgroSPM platform. The architecture is designed to be:

- **Scalable:** Handle 100,000+ concurrent users
- **Reliable:** 99.9% uptime with disaster recovery
- **Secure:** End-to-end encryption and RBAC
- **Performant:** Sub-2-second API responses
- **Cost-effective:** Optimized AWS resource usage
- **Rural-friendly:** Offline-first with low bandwidth requirements

The modular design allows for incremental development and easy maintenance. Each component is independently testable and deployable, enabling continuous delivery and rapid iteration based on user feedback.
