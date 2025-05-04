# Backend Technical Requirements Document: Move with Aman

## 1. Technology Stack

### Core Technologies
- **Runtime Environment**: Node.js (v18.x LTS)
- **Framework**: Express.js
- **Database**: MongoDB with Mongoose ODM
- **Authentication**: JSON Web Tokens (JWT)
- **API Architecture**: RESTful with OpenAPI specification

### Development Tools
- **Package Manager**: npm
- **Process Manager**: PM2 for production
- **Development Server**: Nodemon for hot reloading
- **Testing Framework**: Jest with Supertest
- **Documentation**: Swagger UI / OpenAPI 3.0

## 2. Architecture and Structure

### Project Structure
```
src/
├── config/             # Environment variables and configuration
├── controllers/        # Request handlers
├── middleware/         # Custom middleware functions
├── models/             # Database models and schemas
├── routes/             # API routes
├── services/           # Business logic
├── utils/              # Helper functions and utilities
├── validators/         # Request validation schemas
├── app.js              # Express application setup
└── server.js           # Server entry point
```

### API Structure
- RESTful endpoints following standard HTTP methods
- Resource-based URL structure
- Versioned API paths (e.g., `/api/v1/resources`)
- Consistent response formats

## 3. Database Design

### MongoDB Collections
1. **Users**
   - Authentication information
   - Profile details
   - Role-based permissions

2. **Classes**
   - Class details and descriptions
   - Schedule information
   - Capacity and enrollment counts
   - Instructor references

3. **Events**
   - Event details and descriptions
   - Date and time information
   - Location data
   - Pricing tiers
   - Registration limits

4. **Registrations**
   - User references
   - Class/Event references
   - Payment status
   - Attendance tracking

5. **Blog Posts**
   - Content (title, body, excerpt)
   - Author information
   - Publishing metadata
   - Categories and tags
   - Comments

6. **Media**
   - File metadata
   - Storage references
   - Usage tracking

7. **Contacts**
   - Inquiry submissions
   - Newsletter subscriptions
   - Contact preferences

### Indexing Strategy
- Compound indexes for frequently queried fields
- Text indexes for search functionality
- TTL indexes for temporary data
- Geo indexes for location-based queries

### Data Relationships
- Use of embedded documents for tightly coupled data
- References for many-to-many relationships
- Denormalization strategy for read-heavy operations

## 4. API Endpoints

### Authentication Endpoints
- `POST /api/v1/auth/register`
- `POST /api/v1/auth/login`
- `POST /api/v1/auth/refresh-token`
- `POST /api/v1/auth/forgot-password`
- `POST /api/v1/auth/reset-password`
- `GET /api/v1/auth/me`
- `POST /api/v1/auth/logout`

### User Management Endpoints
- `GET /api/v1/users`
- `GET /api/v1/users/:id`
- `PUT /api/v1/users/:id`
- `DELETE /api/v1/users/:id`
- `GET /api/v1/users/:id/registrations`

### Class & Event Endpoints
- `GET /api/v1/classes`
- `POST /api/v1/classes` (admin)
- `GET /api/v1/classes/:id`
- `PUT /api/v1/classes/:id` (admin)
- `DELETE /api/v1/classes/:id` (admin)
- `GET /api/v1/events`
- `POST /api/v1/events` (admin)
- `GET /api/v1/events/:id`
- `PUT /api/v1/events/:id` (admin)
- `DELETE /api/v1/events/:id` (admin)

### Registration Endpoints
- `POST /api/v1/registrations`
- `GET /api/v1/registrations/:id`
- `PUT /api/v1/registrations/:id` (admin)
- `DELETE /api/v1/registrations/:id`
- `GET /api/v1/classes/:id/registrations` (admin)
- `GET /api/v1/events/:id/registrations` (admin)

### Blog Endpoints
- `GET /api/v1/posts`
- `POST /api/v1/posts` (admin)
- `GET /api/v1/posts/:id`
- `PUT /api/v1/posts/:id` (admin)
- `DELETE /api/v1/posts/:id` (admin)
- `GET /api/v1/categories`
- `POST /api/v1/posts/:id/comments`
- `GET /api/v1/posts/:id/comments`

### Contact Endpoints
- `POST /api/v1/contact`
- `POST /api/v1/newsletter/subscribe`
- `POST /api/v1/newsletter/unsubscribe`

### Admin Endpoints
- `GET /api/v1/admin/dashboard`
- `GET /api/v1/admin/stats`
- `GET /api/v1/admin/reports/registrations`
- `GET /api/v1/admin/reports/revenue`

## 5. Authentication and Authorization

### Authentication Mechanism
- JWT-based authentication
- Access and refresh token strategy
- Secure HTTP-only cookies for token storage
- CSRF protection

### Authorization Levels
1. **Public**: Unauthenticated access
2. **Student**: Registered users with basic privileges
3. **Instructor**: Class-specific management capabilities
4. **Admin**: Full system access

### Security Requirements
- Password hashing with bcrypt
- Rate limiting for sensitive endpoints
- Input validation for all requests
- XSS protection
- Prevention of parameter pollution
- Security headers implementation

## 6. Data Validation

### Validation Strategy
- Request validation using Joi or express-validator
- Schema validation at the MongoDB level
- Custom validators for business rules

### Common Validations
- Email format and uniqueness
- Password strength requirements
- Date and time formats
- Capacity constraints
- Payment information

## 7. Error Handling

### Error Response Format
```json
{
  "status": "error",
  "code": 400,
  "message": "Invalid input data",
  "details": [
    {
      "field": "email",
      "message": "Must be a valid email address"
    }
  ]
}
```

### Error Types
- Validation errors (400)
- Authentication errors (401)
- Authorization errors (403)
- Resource not found (404)
- Method not allowed (405)
- Conflict errors (409)
- Internal server errors (500)

### Error Logging
- Structured error logging with severity levels
- Error aggregation and monitoring
- PII redaction in logs

## 8. Third-Party Integrations

### Payment Processing
- Stripe API integration
- Webhook handling for payment events
- Refund processing
- Invoice generation

### Email Service
- Transactional emails via SendGrid or similar
- Email templates system
- Scheduled newsletter delivery
- Email delivery tracking

### File Storage
- AWS S3 or similar cloud storage
- Image optimization and resizing
- Virus scanning for uploads
- Access control for private files

### External APIs
- Google Maps for location services
- Calendar integration (Google/Apple Calendar)
- Social media posting APIs

## 9. Performance Requirements

### Response Time
- API response time under 200ms for 95% of requests
- Batch operations for data-intensive tasks
- Pagination for list endpoints

### Scaling Strategy
- Horizontal scaling via containerization
- Caching layer with Redis
- Database read replicas for scaling reads
- Sharding strategy for future growth

### Caching
- Response caching for public endpoints
- Data caching for expensive operations
- Cache invalidation strategy
- ETags for conditional requests

## 10. Testing Requirements

### Testing Levels
- Unit testing for services and utilities
- Integration testing for API endpoints
- Load testing for performance validation
- Security testing for vulnerabilities

### Test Coverage
- Minimum 80% code coverage
- Critical paths coverage of 100%
- Regression test suite

### Testing Environment
- Isolated testing database
- Mock external services
- CI/CD integration

## 11. Logging and Monitoring

### Logging Strategy
- Structured logging (JSON format)
- Log levels (debug, info, warn, error)
- Request/response logging with sampling
- Audit logging for sensitive operations

### Monitoring Requirements
- Health check endpoints
- Performance metrics collection
- Real-time error alerting
- Usage analytics

## 12. Deployment Requirements

### Environment Configuration
- Environment-specific configuration
- Secret management with environment variables
- Feature flags for gradual rollout

### Deployment Pipeline
- Automated testing in CI
- Staging deployment for verification
- Blue-green deployment strategy
- Rollback capabilities

### Production Environment
- Load balancing
- Auto-scaling configuration
- Database backup strategy
- Disaster recovery plan

## 13. Documentation Requirements

### API Documentation
- OpenAPI/Swagger specification
- Interactive API explorer
- Example requests and responses
- Authentication instructions

### Developer Documentation
- Setup and installation guide
- Architecture overview
- Database schema documentation
- Contribution guidelines

## 14. Maintenance Plan

### Updates and Patches
- Security patch schedule
- Dependency update policy
- Database maintenance windows

### Backup Strategy
- Daily automated backups
- Point-in-time recovery capability
- Cross-region backup storage
- Backup restoration testing

## 15. Compliance Requirements

### Data Protection
- GDPR compliance for EU users
- Data portability support
- Right to be forgotten implementation
- Data minimization practices

### Accessibility
- WCAG 2.1 AA backend considerations
- Semantic response structures
- Proper error messaging

### Security Compliance
- OWASP Top 10 protection
- Regular security audits
- Penetration testing schedule
- Vulnerability disclosure policy