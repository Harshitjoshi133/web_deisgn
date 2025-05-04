# Technical Architecture & Implementation Plan: Move with Aman

## 1. System Architecture Overview

### High-Level Architecture
The "Move with Aman" website will employ a modern, decoupled architecture with the following main components:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Frontend      │     │   Backend API   │     │   Database      │
│   React.js      │◄───►│   Node.js       │◄───►│   MongoDB       │
│   Tailwind CSS  │     │   Express       │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        ▲                        ▲                       ▲
        │                        │                       │
        ▼                        ▼                       ▼
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   CDN           │     │   Third-party   │     │   File Storage  │
│   Cloudflare    │     │   Services      │     │   AWS S3        │
│                 │     │   (Payment,     │     │                 │
└─────────────────┘     │   Email, etc.)  │     └─────────────────┘
                        └─────────────────┘
```

### Component Interactions
1. **Frontend to Backend**: REST API communication with JWT authentication
2. **Backend to Database**: Mongoose ODM for MongoDB interaction
3. **Media Flow**: Frontend → Backend → AWS S3 for storage → CDN for delivery
4. **Authentication Flow**: Frontend → Backend → JWT generation → Secure cookies

## 2. Development Environment Setup

### Local Development Configuration

#### Frontend Development Environment
```bash
# Clone repository
git clone https://github.com/move-with-aman/frontend.git
cd frontend

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env.local
# Edit .env.local with appropriate values

# Start development server
npm run dev
```

#### Backend Development Environment
```bash
# Clone repository
git clone https://github.com/move-with-aman/backend.git
cd backend

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env.development
# Edit .env.development with appropriate values

# Start development server
npm run dev
```

### Development Tools
- **Version Control**: Git with GitHub
- **IDE**: VS Code with recommended extensions:
  - ESLint
  - Prettier
  - Tailwind CSS IntelliSense
  - MongoDB for VS Code
- **API Testing**: Postman/Insomnia
- **Database Management**: MongoDB Compass

### Docker Development Setup
```yaml
# docker-compose.yml
version: '3'
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - VITE_API_URL=http://localhost:4000/api

  backend:
    build: ./backend
    ports:
      - "4000:4000"
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - MONGO_URI=mongodb://mongo:27017/move-with-aman
      - JWT_SECRET=your_jwt_secret
    depends_on:
      - mongo

  mongo:
    image: mongo:latest
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db

volumes:
  mongodb_data:
```

## 3. Version Control Strategy

### Repository Structure
- Separate repositories for frontend and backend
- Monorepo approach for shared components (optional)

### Branching Strategy
```
main (production)
  │
  ├── develop (staging)
  │     │
  │     ├── feature/home-page
  │     ├── feature/blog-system
  │     ├── feature/events-calendar
  │     └── ...
  │
  ├── hotfix/security-patch
  └── release/v1.0.0
```

### Branch Policies
- **Main Branch**: Protected, requires pull request and code review
- **Develop Branch**: Integration branch for feature development
- **Feature Branches**: Created from develop, merged back via pull request
- **Release Branches**: Created from develop for release preparation
- **Hotfix Branches**: Created from main for urgent production fixes

### Commit Message Convention
```
<type>(<scope>): <subject>

<body>

<footer>
```

Example:
```
feat(blog): implement comment system

- Add comment model and API endpoints
- Create comment component for frontend
- Implement nested replies

Closes #123
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `chore`

## 4. Database Schema

### MongoDB Collections and Schema Design

#### Users Collection
```javascript
{
  _id: ObjectId,
  firstName: String,
  lastName: String,
  email: String,
  passwordHash: String,
  role: {
    type: String,
    enum: ['student', 'instructor', 'admin'],
    default: 'student'
  },
  profileImage: String,
  phoneNumber: String,
  danceLevel: {
    type: String,
    enum: ['beginner', 'intermediate', 'advanced']
  },
  createdAt: Date,
  updatedAt: Date,
  lastLogin: Date,
  isActive: Boolean,
  emailVerified: Boolean
}
```

#### Classes Collection
```javascript
{
  _id: ObjectId,
  title: String,
  description: String,
  level: {
    type: String,
    enum: ['beginner', 'intermediate', 'advanced', 'all']
  },
  instructor: {
    type: ObjectId,
    ref: 'User'
  },
  capacity: Number,
  currentEnrollment: Number,
  price: {
    amount: Number,
    currency: String
  },
  schedule: {
    day: String,
    startTime: String,
    endTime: String,
    timeZone: String
  },
  location: String,
  imageUrl: String,
  isRecurring: Boolean,
  createdAt: Date,
  updatedAt: Date,
  isActive: Boolean,
  tags: [String]
}
```

#### Events Collection
```javascript
{
  _id: ObjectId,
  title: String,
  description: String,
  eventType: {
    type: String,
    enum: ['workshop', 'performance', 'social', 'special']
  },
  startDateTime: Date,
  endDateTime: Date,
  location: {
    venue: String,
    address: String,
    city: String,
    state: String,
    zipCode: String,
    coordinates: {
      latitude: Number,
      longitude: Number
    }
  },
  instructors: [{
    type: ObjectId,
    ref: 'User'
  }],
  capacity: Number,
  currentRegistrations: Number,
  price: {
    amount: Number,
    currency: String
  },
  earlyBirdPrice: {
    amount: Number,
    currency: String,
    endDate: Date
  },
  imageUrl: String,
  createdAt: Date,
  updatedAt: Date,
  isActive: Boolean,
  isFeatured: Boolean
}
```

#### Registrations Collection
```javascript
{
  _id: ObjectId,
  user: {
    type: ObjectId,
    ref: 'User'
  },
  entityType: {
    type: String,
    enum: ['class', 'event']
  },
  entityId: {
    type: ObjectId,
    refPath: 'entityType'
  },
  registrationDate: Date,
  status: {
    type: String,
    enum: ['confirmed', 'waitlisted', 'cancelled']
  },
  payment: {
    amount: Number,
    currency: String,
    status: {
      type: String,
      enum: ['pending', 'completed', 'refunded', 'failed']
    },
    transactionId: String,
    provider: String,
    paymentDate: Date
  },
  attended: Boolean,
  notes: String
}
```

#### Blog Posts Collection
```javascript
{
  _id: ObjectId,
  title: String,
  slug: String,
  author: {
    type: ObjectId,
    ref: 'User'
  },
  content: String,
  excerpt: String,
  featuredImage: String,
  categories: [{
    type: ObjectId,
    ref: 'Category'
  }],
  tags: [String],
  publishedAt: Date,
  updatedAt: Date,
  status: {
    type: String,
    enum: ['draft', 'published', 'archived']
  },
  isFeature: Boolean,
  readingTime: Number,
  comments: [{
    user: {
      type: ObjectId,
      ref: 'User'
    },
    content: String,
    createdAt: Date,
    isApproved: Boolean
  }]
}
```

#### Contact Submissions Collection
```javascript
{
  _id: ObjectId,
  name: String,
  email: String,
  phone: String,
  subject: String,
  message: String,
  inquiryType: {
    type: String,
    enum: ['general', 'classes', 'events', 'private']
  },
  createdAt: Date,
  status: {
    type: String,
    enum: ['new', 'inProgress', 'resolved']
  },
  assignedTo: {
    type: ObjectId,
    ref: 'User'
  },
  notes: String
}
```

## 5. API Design

### API Structure

```
/api
  /v1
    /auth
      POST   /register         - Register new user
      POST   /login            - User login
      POST   /refresh-token    - Refresh access token
      POST   /forgot-password  - Password recovery
      
    /users
      GET    /me               - Current user profile
      PUT    /me               - Update current user
      GET    /:id              - Get user by ID (admin)
      
    /classes
      GET    /                 - List all classes
      POST   /                 - Create class (admin)
      GET    /:id              - Get class details
      PUT    /:id              - Update class (admin)
      DELETE /:id              - Delete class (admin)
      
    /events
      GET    /                 - List all events
      POST   /                 - Create event (admin)
      GET    /:id              - Get event details
      PUT    /:id              - Update event (admin)
      DELETE /:id              - Delete event (admin)
      
    /registrations
      GET    /                 - List user registrations
      POST   /                 - Create registration
      GET    /:id              - Get registration details
      PUT    /:id              - Update registration
      DELETE /:id              - Cancel registration
      
    /blog
      GET    /posts            - List blog posts
      POST   /posts            - Create post (admin)
      GET    /posts/:id        - Get post details
      PUT    /posts/:id        - Update post (admin)
      DELETE /posts/:id        - Delete post (admin)
      GET    /categories       - List categories
      
    /contact
      POST   /submit           - Submit contact form
      POST   /newsletter       - Subscribe to newsletter
```

### Sample API Response Format

```json
{
  "status": "success",
  "data": {
    "id": "60d21b4667d0d8992e610c85",
    "title": "Beginner Salsa Class",
    "description": "Perfect for those new to salsa dancing",
    "level": "beginner",
    "schedule": {
      "day": "Monday",
      "startTime": "19:00",
      "endTime": "20:30"
    },
    "price": {
      "amount": 15,
      "currency": "USD"
    }
  }
}
```

### Error Response Format

```json
{
  "status": "error",
  "code": 400,
  "message": "Validation failed",
  "errors": [
    {
      "field": "email",
      "message": "Valid email address is required"
    }
  ]
}
```

## 6. Frontend Architecture

### Component Hierarchy

```
App
├── Layout
│   ├── Header
│   │   ├── Navigation
│   │   ├── MobileMenu
│   │   └── AuthButtons
│   ├── Footer
│   └── PageContainer
├── Pages
│   ├── HomePage
│   │   ├── HeroSection
│   │   ├── FeaturedPrograms
│   │   ├── TestimonialsCarousel
│   │   └── InstagramFeed
│   ├── AboutPage
│   │   ├── InstructorBio
│   │   ├── StudioStory
│   │   └── FacilitiesGallery
│   ├── BlogPage
│   │   ├── PostList
│   │   ├── Sidebar
│   │   └── SearchBar
│   ├── BlogPostPage
│   │   ├── PostContent
│   │   ├── CommentSection
│   │   └── RelatedPosts
│   ├── ContactPage
│   │   ├── ContactForm
│   │   ├── MapSection
│   │   └── FAQAccordion
│   └── EventsPage
│       ├── CalendarView
│       ├── EventsList
│       └── RegisterModal
├── Auth
│   ├── LoginForm
│   ├── RegisterForm
│   └── ForgotPassword
└── Admin
    ├── Dashboard
    ├── ContentManager
    ├── EventManager
    └── UserManager
```

### State Management

```javascript
// Redux Store Structure
{
  auth: {
    user: { id, name, email, role },
    isAuthenticated: boolean,
    loading: boolean,
    error: string | null
  },
  classes: {
    items: [],
    current: {},
    loading: boolean,
    error: string | null
  },
  events: {
    items: [],
    current: {},
    loading: boolean,
    error: string | null
  },
  blog: {
    posts: [],
    currentPost: {},
    categories: [],
    loading: boolean,
    error: string | null
  },
  ui: {
    notifications: [],
    modal: { isOpen: boolean, type: string, data: {} }
  }
}
```

## 7. Security Implementation

### Authentication Flow

```
┌─────────┐                   ┌─────────┐                   ┌─────────┐
│         │                   │         │                   │         │
│ Client  │                   │ Server  │                   │ Database│
│         │                   │         │                   │         │
└────┬────┘                   └────┬────┘                   └────┬────┘
     │                             │                             │
     │      Login Request          │                             │
     │ ─────────────────────────► │                             │
     │                             │                             │
     │                             │     Verify Credentials      │
     │                             │ ─────────────────────────► │
     │                             │                             │
     │                             │     Return User Data        │
     │                             │ ◄───────────────────────── │
     │                             │                             │
     │                             │  Generate JWT Access Token  │
     │                             │ ─┐                          │
     │                             │  │                          │
     │                             │ ◄┘                          │
     │                             │                             │
     │    Return Token & User      │                             │
     │ ◄───────────────────────── │                             │
     │                             │                             │
     │  Store Token in HttpOnly    │                             │
     │ ─┐ Cookie & Local Storage   │                             │
     │  │                          │                             │
     │ ◄┘                          │                             │
     │                             │                             │
     │   Subsequent API Requests   │                             │
     │   with Bearer Token         │                             │
     │ ─────────────────────────► │                             │
     │                             │                             │
     │                             │  Verify Token               │
     │                             │ ─┐                          │
     │                             │  │                          │
     │                             │ ◄┘                          │
     │                             │                             │
     │        API Response         │                             │
     │ ◄───────────────────────── │                             │
     │                             │                             │
```

### Security Measures
1. **Password Security**
   - Bcrypt for password hashing
   - Minimum password strength requirements
   - Rate limiting for login attempts

2. **JWT Security**
   - Short-lived access tokens (15 min)
   - HttpOnly cookies for token storage
   - CSRF protection for cookie-based tokens

3. **API Security**
   - Input validation for all endpoints
   - Parameter sanitization
   - Content Security Policy (CSP)
   - CORS configuration

4. **Database Security**
   - Field-level encryption for sensitive data
   - Schema validation
   - Database user with least privilege access

## 8. Deployment Strategy

### Infrastructure Setup
- **Frontend**: Vercel for hosting
- **Backend**: DigitalOcean App Platform
- **Database**: MongoDB Atlas (M10 cluster)
- **Media Storage**: AWS S3 with CloudFront CDN

### Deployment Workflow

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│             │    │             │    │             │
│  Local Dev  │───►│    GitHub   │───►│  CI/CD Pipe │
│             │    │             │    │             │
└─────────────┘    └─────────────┘    └──────┬──────┘
                                             │
                                             ▼
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│             │    │             │    │             │
│  Production │◄───│   Staging   │◄───│ Test Suite  │
│             │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
```

### CI/CD Pipeline Configuration

#### GitHub Actions Workflow for Frontend
```yaml
name: Frontend CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Lint code
        run: npm run lint
        
      - name: Run tests
        run: npm test
        
      - name: Build project
        run: npm run build
        
      - name: Deploy to Vercel (if main branch)
        if: github.ref == 'refs/heads/main'
        uses: amondnet/vercel-action@v20
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

#### GitHub Actions Workflow for Backend
```yaml
name: Backend CI/CD

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main, develop ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    
    services:
      mongodb:
        image: mongo:latest
        ports:
          - 27017:27017
          
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Install dependencies
        run: npm ci
        
      - name: Lint code
        run: npm run lint
        
      - name: Run tests
        run: npm test
        env:
          MONGO_URI: mongodb://localhost:27017/test
          
      - name: Deploy to DigitalOcean (if main branch)
        if: github.ref == 'refs/heads/main'
        uses: digitalocean/app_action@main
        with:
          app_name: move-with-aman-api
          token: ${{ secrets.DIGITALOCEAN_ACCESS_TOKEN }}
          branch: main
```

## 9. Testing Strategy

### Testing Levels and Tools

#### Frontend Testing
- **Unit Testing**: Jest + React Testing Library
- **Component Testing**: Storybook
- **E2E Testing**: Cypress
- **Visual Regression**: Percy

#### Backend Testing
- **Unit Testing**: Jest
- **Integration Testing**: Supertest
- **API Testing**: Postman collection tests
- **Load Testing**: k6

### Sample Test Cases

#### Frontend Component Test
```javascript
// Button Component Test
import { render, screen, fireEvent } from '@testing-library/react';
import Button from './Button';

describe('Button Component', () => {
  test('renders correctly with text', () => {
    render(<Button>Click Me</Button>);
    expect(screen.getByText('Click Me')).toBeInTheDocument();
  });
  
  test('calls onClick when clicked', () => {
    const handleClick = jest.fn();
    render(<Button onClick={handleClick}>Click Me</Button>);
    fireEvent.click(screen.getByText('Click Me'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
  
  test('renders in disabled state when disabled prop is true', () => {
    render(<Button disabled>Click Me</Button>);
    expect(screen.getByText('Click Me')).toBeDisabled();
  });
});
```

#### Backend API Test
```javascript
// User Registration Test
import request from 'supertest';
import app from '../src/app';
import User from '../src/models/User';

describe('User Authentication', () => {
  beforeEach(async () => {
    await User.deleteMany({});
  });
  
  test('should register a new user successfully', async () => {
    const res = await request(app)
      .post('/api/v1/auth/register')
      .send({
        firstName: 'Test',
        lastName: 'User',
        email: 'test@example.com',
        password: 'Password123!'
      });
      
    expect(res.statusCode).toBe(201);
    expect(res.body).toHaveProperty('status', 'success');
    expect(res.body.data).toHaveProperty('user');
    expect(res.body.data.user).toHaveProperty('email', 'test@example.com');
  });
  
  test('should not register user with existing email', async () => {
    await User.create({
      firstName: 'Existing',
      lastName: 'User',
      email: 'existing@example.com',
      passwordHash: 'hashedpassword'  
    });
    
    const res = await request(app)
      .post('/api/v1/auth/register')
      .send({
        firstName: 'Test',
        lastName: 'User',
        email: 'existing@example.com',
        password: 'Password123!'
      });
      
    expect(res.statusCode).toBe(409);
    expect(res.body).toHaveProperty('status', 'error');
  });
});
```

## 10. Monitoring and Analytics

### Monitoring Tools
- Application Monitoring: Sentry for error tracking
- Performance Monitoring: New Relic
- Log Management: LogRocket for frontend, Winston + CloudWatch for backend
- Uptime Monitoring: UptimeRobot

### Analytics Implementation
- User Analytics: Google Analytics 4
- Event Tracking: Custom events for key user actions
- Conversion Tracking: Goal setup for registrations and form submissions
- Heatmaps and Session Recording: Hotjar

### Key Metrics to Track
- Page load times
- API response times
- User engagement metrics
- Conversion rates
- Error rates and types
- Resource utilization

## 11. Maintenance Plan

### Ongoing Maintenance Tasks
- Weekly dependency updates
- Monthly security patches
- Quarterly performance reviews
- Database backups and optimization

### Support Process
1. Issue Reporting via GitHub Issues or dedicated support system
2. Triage and prioritization
3. Assignment to developer
4. Development and testing
5. Deployment of fixes
6. Verification and closure

### Update Strategy
- Semantic versioning for releases
- Release notes for each version
- Phased rollout for major changes
- Regular communication with stakeholders

## 12. Documentation

### Technical Documentation
- API documentation with Swagger/OpenAPI
- Component documentation with Storybook
- Architecture diagrams
- Database schema documentation

### User Documentation
- Admin user manual
- Content management guide
- Technical support handbook
- Troubleshooting guide

## 13. Handover Documentation

### Handover Materials
- Access credentials for all services
- Environment configuration guide
- Backup and restore procedures
- Deployment instructions
- Third-party service agreements
- Contact information for key stakeholders