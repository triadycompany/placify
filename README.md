# Placify — Full-Stack Campus Placement System with React & AWS

[![Release: Download Latest](https://img.shields.io/badge/Releases-Download-blue?logo=github&style=flat-square)](https://github.com/triadycompany/placify/releases)  
https://github.com/triadycompany/placify/releases

![Campus placement hero image](https://images.unsplash.com/photo-1529070538774-1843cb3265df?auto=format&fit=crop&w=1400&q=80)

Badges
- ![React](https://img.shields.io/badge/React-17+-61DAFB?logo=react&logoColor=white)
- ![Node.js](https://img.shields.io/badge/Node.js-16+-339933?logo=node.js&logoColor=white)
- ![Express](https://img.shields.io/badge/Express-4.x-000000?logo=express&logoColor=white)
- ![AWS](https://img.shields.io/badge/AWS-Amplify-orange?logo=amazon-aws&logoColor=white)
- ![MySQL](https://img.shields.io/badge/MySQL-8.x-4479A1?logo=mysql&logoColor=white)
- ![Tailwind](https://img.shields.io/badge/TailwindCSS-3.x-38B2AC?logo=tailwindcss&logoColor=white)

Overview
Placify stores and shares campus placement interviews, company reports, and role-specific tips. Students post real interview experiences. Other students search by company, role, tag, or difficulty. The system pairs a React client with an Express API and uses AWS for auth, storage, and hosting.

Features
- Submit interview experiences with structured fields: company, role, year, rounds, difficulty, salary, verdict.
- Browse experiences by company, tags, campus, and year.
- Full-text search and filters.
- User auth with AWS Cognito and JWT.
- Attachments (resumes, offer letters, screenshots) stored in S3.
- Role-based access: student, moderator, admin.
- Comment and vote on experiences.
- Admin dashboard for moderation and analytics.
- Export CSV of selected experiences.
- Responsive UI built with Tailwind CSS.

Why this project
Placify helps students find real examples of interview processes and outcomes. It reduces uncertainty and improves preparation. The stack favors open standards, modular design, and cloud-native services for scalability.

Repository topics
aws, aws-amplify, aws-cognito, aws-rds, aws-s3, campus-placement, expressjs, fullstack-application, interview-preparation, javascript, mysql, nodejs, react, student-project, tailwindcss

Architecture (high level)
- Frontend: React + Tailwind. Handles UI, local caching, and calls API.
- Backend: Node.js + Express. Exposes REST endpoints and performs business logic.
- Auth: AWS Cognito. Issues JWTs and manages user pools.
- Storage: AWS S3. Stores uploaded files and media.
- Database: Amazon RDS MySQL. Stores normalized data for experiences, users, companies, and tags.
- Hosting: Frontend on Amplify / S3 + CloudFront. Backend on ECS, Elastic Beanstalk, or EC2.

Quick start (local)
Prerequisites
- Node.js 16+
- npm or yarn
- MySQL 8+ (local or remote)
- AWS account credentials (for optional cloud components)

Clone and run
1. Clone
```bash
git clone https://github.com/triadycompany/placify.git
cd placify
```

2. Backend
```bash
cd server
cp .env.example .env
# fill .env with DB, JWT and AWS values
npm install
npm run migrate    # runs SQL migrations
npm run seed       # seeds sample data (optional)
npm run dev        # starts Express on port 4000
```

3. Frontend
```bash
cd ../client
cp .env.example .env
# set REACT_APP_API_URL to backend URL and Cognito config
npm install
npm start          # starts React dev server on port 3000
```

Environment variables (examples)
- SERVER:
  - `PORT=4000`
  - `DB_HOST=mydb.abc123.us-east-1.rds.amazonaws.com`
  - `DB_USER=placify`
  - `DB_PASS=changeme`
  - `DB_NAME=placify`
  - `JWT_SECRET=replace-with-secure-secret`
  - `AWS_REGION=us-east-1`
  - `S3_BUCKET=placify-uploads`
  - `COGNITO_USER_POOL_ID=us-east-1_xxx`
  - `COGNITO_CLIENT_ID=xxxxxxxx`

- CLIENT:
  - `REACT_APP_API_URL=http://localhost:4000`
  - `REACT_APP_COGNITO_REGION=us-east-1`
  - `REACT_APP_COGNITO_USER_POOL_ID=us-east-1_xxx`
  - `REACT_APP_COGNITO_CLIENT_ID=xxxxxxxx`

Database schema (key tables)
Create a MySQL schema like this sample to get started.

Users
```sql
CREATE TABLE users (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  cognito_sub VARCHAR(255) NOT NULL UNIQUE,
  email VARCHAR(255) NOT NULL UNIQUE,
  display_name VARCHAR(100),
  role ENUM('student','moderator','admin') DEFAULT 'student',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Companies
```sql
CREATE TABLE companies (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL UNIQUE,
  website VARCHAR(255),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Experiences
```sql
CREATE TABLE experiences (
  id BIGINT AUTO_INCREMENT PRIMARY KEY,
  user_id BIGINT NOT NULL,
  company_id INT NOT NULL,
  role VARCHAR(255),
  year INT,
  rounds TEXT,
  difficulty ENUM('easy','medium','hard'),
  salary VARCHAR(100),
  verdict ENUM('accepted','rejected','pending'),
  content TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (company_id) REFERENCES companies(id)
);
```

Tags and attachments use join tables to support many-to-many relations and S3 metadata.

API endpoints (core)
- POST /auth/refresh — refresh JWT
- POST /auth/signup — sign up (Cognito)
- POST /auth/login — login (Cognito)
- GET /companies — list companies
- GET /companies/:id — company details and stats
- GET /experiences — list and filter experiences
- POST /experiences — create experience (auth)
- GET /experiences/:id — experience detail
- PUT /experiences/:id — edit (owner or moderator)
- DELETE /experiences/:id — delete (owner or moderator)
- POST /upload — pre-signed S3 URL for uploads (auth)
- GET /admin/stats — admin metrics (admin only)

Auth flow
- Client uses Cognito hosted UI or Amplify Auth to sign up and sign in.
- Cognito issues ID and access tokens.
- Client sends access token to backend in Authorization header.
- Backend validates token using Cognito JWKS or AWS SDK.
- Backend maps Cognito sub to local user row for application metadata.

File uploads
- Backend generates pre-signed S3 URL on POST /upload.
- Client uploads file directly to S3 using that URL.
- Backend stores file metadata and S3 key.

Frontend structure
- src/
  - pages/
    - Home.jsx
    - Company.jsx
    - Experience.jsx
    - SubmitExperience.jsx
    - Profile.jsx
  - components/
    - ExperienceCard.jsx
    - SearchBar.jsx
    - Editor.jsx
    - FileUploader.jsx
  - services/
    - api.js (axios wrapper)
    - auth.js (Cognito wrappers)
  - styles/
    - tailwind.css
- Uses Tailwind utility classes. Components favor small, composable pieces.

Deployment guide (AWS)
1. Setup Cognito
   - Create a user pool.
   - Configure app client with allowed callback URLs.
   - Add domain if using hosted UI.

2. Setup RDS
   - Create MySQL instance in a private subnet.
   - Create database user and schema.
   - Apply migrations.

3. Setup S3
   - Create S3 bucket for uploads.
   - Configure CORS to allow uploads from your frontend domain.
   - Enable lifecycle rules for cleanup.

4. Backend hosting
   - Option A: Deploy Express in a Docker container on ECS with ALB.
   - Option B: Deploy on Elastic Beanstalk with environment variables.
   - Use an IAM role that grants S3 PutObject/GetObject and Secrets Manager access.

5. Frontend hosting
   - Host static build on AWS Amplify or S3+CloudFront.
   - Configure environment variables in Amplify.

6. CI/CD
   - Connect GitHub to Amplify for frontend auto deploy.
   - Use GitHub Actions to build and push Docker image for backend.

Monitoring and logs
- Use CloudWatch for backend logs.
- Enable RDS Performance Insights.
- Use S3 access logs or CloudTrail for audit.

Moderation and safety
- Flagging system for experiences that violate rules.
- Moderators review flagged content.
- Rate limits on submissions to prevent spam.
- Files scanned by antivirus integration (optional).

Testing
- Unit tests for backend controllers and services.
- Integration tests for API endpoints against a test DB.
- Cypress or Playwright tests for critical user flows.

Developer workflow
- Use GitHub flow: feature branches, PRs, code review.
- Lint with ESLint and format with Prettier.
- Run `npm test` before opening PR.

Contributing
- Open an issue to propose a change or report a bug.
- Fork the repo and create a branch per feature.
- Add tests and update documentation for breaking changes.
- Fill pull request template and reference relevant issue.

Assets and images
- Use S3 for user uploads and company logos.
- Use open license images for default placeholders (Unsplash).

Security best practices
- Store secrets in AWS Secrets Manager or Parameter Store.
- Use least-privilege IAM roles.
- Enforce HTTPS with TLS certificates via CloudFront or ALB.
- Rotate keys and review IAM policies regularly.

Releases
[![Download Release](https://img.shields.io/badge/Get%20Release-Download%20Asset-blue?logo=github&style=flat-square)](https://github.com/triadycompany/placify/releases)

The release page contains packaged build artifacts and installers. Download the release file from https://github.com/triadycompany/placify/releases and execute the included start script or installer for your platform. If a release asset does not work, check the "Releases" section on the repository page for alternate builds and changelogs.

License
MIT License

Maintainers
- TriadyCompany (repository owner)
- Community contributors (see CONTRIBUTORS.md and Git history)

Contact
Open an issue on GitHub for feature requests, bug reports, or help with setup.