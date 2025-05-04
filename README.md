# Move with Aman - Dance Studio Website

A full-stack web application for a professional salsa dance studio, built with React.js and Node.js.

## Tech Stack

### Frontend
- React.js 18+
- Tailwind CSS
- Redux Toolkit
- React Router

### Backend
- Node.js & Express.js
- MongoDB with Mongoose
- JWT Authentication
- REST API

## Key Features
- Class & Event Management
- Student Registration System
- Blog Platform
- Admin Dashboard
- Payment Integration
- Responsive Design

## Getting Started

### Prerequisites
- Node.js v18+
- MongoDB
- npm/yarn

### Installation
```bash
# Clone repositories
git clone https://github.com/move-with-aman/frontend.git
git clone https://github.com/move-with-aman/backend.git

# Frontend setup
cd frontend
npm install
npm run dev

# Backend setup
cd backend
npm install
npm run dev
```

## Environment Variables
Create `.env` files in both frontend and backend directories:

```env
# Frontend
VITE_API_URL=http://localhost:4000/api

# Backend
MONGO_URI=your_mongodb_uri
JWT_SECRET=your_jwt_secret
```

## Documentation
- [Frontend Technical Specs](frontend.md)
- [Backend Technical Specs](backend.md)
- [Project Requirements](requirements.md)
- [Implementation Timeline](timeline.md)
- [Technical Architecture](tech_architecture.md)

## License
MIT