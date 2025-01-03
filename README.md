# System Design for "Find Old Classmates" App

## 1. Requirements Analysis

### Functional Requirements
- **User Registration and Login**
  - Users can register with an email
  - Login support for email

- **Profile Management**
  - Fill in school name (mandatory), class name (optional), and education stage (e.g., elementary, middle school, high school, college, graduate school).
  - Update personal details.

- **Search Functionality**
  - Search for users by school name, education stage, and class name.

- **Privacy Control**
  - Users can choose which information (e.g., email, phone, bio) to make public.

- **Message Board**
  - Users can leave public messages on school-specific boards
  - Users receive email notifications when someone posts on their school's board
  - Messages include timestamp, author name, and optional contact info

### Non-functional Requirements
- **Performance**: Support high concurrent access.
- **Scalability**: Allow easy addition of new features.
- **Security**: Encrypt sensitive data and provide privacy controls.
- **Cross-Platform Support**: Accessible on iOS, Android, and web platforms.

---

## 2. System Architecture Design

### Tech Stack
| **Layer**       | **Technology**                                               |
|------------------|-------------------------------------------------------------|
| **Frontend**     | SwiftUI (iOS), Jetpack Compose (Android), React (Web).       |
| **Backend**      | Go (framework: Gin or Fiber for simplicity and performance). |
| **Database**     | PostgreSQL (relational database for structured data).        |
| **Caching**      | Redis (to speed up frequently accessed searches).            |
| **Authentication** | OAuth 2.0 or JWT for secure login and API access.           |
| **Storage**      | AWS S3 or Alibaba Cloud OSS for static file storage.         |
| **Deployment**   | Docker + Kubernetes, hosted on AWS or GCP.                  |

---

### System Architecture Diagram
```text
Client (iOS/Android/Web)
       │
       └───────>API 
                 │
       ┌─────────┴──────────┐
       │                    │
Customer Service     Search Service
（register/login）   （search by school/stage）
       │                    │
       └──────> DB（PostgreSQL）<── Cache（Redis）
```

---

## 3. Database Design

### Tables

#### Users Table (`users`)
| Field Name       | Data Type       | Description                        |
|------------------|-----------------|------------------------------------|
| `id`             | UUID           | Unique user identifier.            |
| `name`           | VARCHAR(100)   | User's name.                       |
| `email`          | VARCHAR(255)   | Email address (optional).          |
| `phone`          | VARCHAR(15)    | Phone number (optional).           |
| `password_hash`  | VARCHAR(255)   | Encrypted password (e.g., bcrypt). |
| `created_at`     | TIMESTAMP      | Account creation timestamp.        |

#### Social Media Table (`social_media`)
| Field Name       | Data Type       | Description                        |
|------------------|-----------------|------------------------------------|
| `id`             | UUID           | Unique identifier for the social media record. |
| `user_id`        | UUID           | Foreign key referencing the users table.       |
| `platform`       | ENUM           | The social media platform (e.g., Instagram). |
| `account_name`   | VARCHAR(255)   | The user's account/username on the platform. |
| `created_at`     | TIMESTAMP      | Record creation timestamp.                    |

#### Education Table (`education`)
| Field Name       | Data Type       | Description                        |
|------------------|-----------------|------------------------------------|
| `id`             | UUID           | Primary key.                       |
| `user_id`        | UUID           | Foreign key linking to `users`.    |
| `school_name`    | VARCHAR(255)   | Name of the school.                |
| `country`        | VARCHAR(100)   | Country where school is located.   |
| `city`          | VARCHAR(100)    | City where school is located.      |
| `stage`          | ENUM           | Education stage (e.g., high school).|
| `class_name`     | VARCHAR(50)    | Class name (optional).             |
| `created_at`     | TIMESTAMP      | Record creation timestamp.         |

#### Message Board Table (`messages`)
| Field Name       | Data Type       | Description                        |
|------------------|-----------------|------------------------------------|
| `id`             | UUID           | Primary key                        |
| `school_id`      | UUID           | Foreign key to education.id        |
| `author_id`      | UUID           | Foreign key to users.id            |
| `content`        | TEXT           | Message content                    |
| `is_public`      | BOOLEAN        | Whether message is public          |
| `created_at`     | TIMESTAMP      | Message creation timestamp         |

---

## 4. Functional Module Design

### User Registration and Login
1. **Workflow**:
   - Users register by providing at least one contact method (email, phone, or social media).
   - Passwords are hashed before storing (e.g., bcrypt).
   - OAuth login (e.g., Google, Facebook) is supported.

2. **Security Measures**:
   - Generate JWT tokens upon login (valid for 7 days).
   - Encrypt sensitive information (e.g., email).

### Profile Management
- Users can update school, stage, and class details.
- Input validation:
  - School name is required.
  - Stage must be a predefined enum value.

### Search Functionality
1. **Query Logic**:
   - Allow search by school, stage, country, city, and optional class name.
   - Use Redis to cache popular search results by location.

2. **Privacy and Security**:
   - Display only public information by default.
   - Users control visibility settings.

### Message Board Functionality
1. **Posting Messages**:
   - Users can post messages to their school's board
   - Messages are linked to specific schools and education stages
   - Optional: Include contact information in messages

2. **Notification System**:
   - Email notifications sent to users when new messages appear on their school's board
   - Users can opt out of notifications
   - Rate limiting on notifications to prevent spam

3. **Privacy Controls**:
   - Messages can be public or visible only to verified school members
   - Users can delete their own messages
   - School-specific visibility settings

---

## 5. Security Design

1. **Data Protection**
   - Use HTTPS for secure data transmission.
   - Encrypt sensitive user data (e.g., phone, email).

2. **Authentication**
   - JWT for secure session handling.
   - Validate JWT tokens for API access.

3. **Privacy Controls**
   - Users can choose what information is visible to others.

---

## 6. Performance Optimization

1. **Database Optimization**
   - Add indexes to `school_name`, `stage`, `country`, and `city` for faster queries.
   - Consider composite indexes for common search patterns (e.g., country + city).

2. **Caching**
   - Cache popular queries (e.g., specific schools) in Redis.
   - Cache user profiles for frequently accessed data.

3. **Load Balancing**
   - Use Nginx or AWS ELB to distribute traffic.

---

## 7. Development Roadmap

### Phase 1: Prototype Development
- Implement core features: registration, login, profile management, and basic search.
- Validate backend logic using PostgreSQL and Redis.

### Phase 2: Frontend Development
- Build mobile clients with SwiftUI (iOS) and Jetpack Compose (Android).
- Develop the web client using React.

### Phase 3: Testing and Optimization
- Perform unit and integration testing.
- Conduct security testing (e.g., vulnerability scans).

### Phase 4: Deployment and Iteration
- Deploy on a cloud platform.
- Gather user feedback for iterative improvements.