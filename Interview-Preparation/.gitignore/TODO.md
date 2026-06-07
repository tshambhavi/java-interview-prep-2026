⬜ Add certifications (in progress)
⬜ Fix GitHub + add link
⬜ Add one Java project to Projects section

Priority Certifications for Your Profile
🔴 High Priority (Do These First)
1. Docker Essentials — IBM (Free)

Platform: Cognitive Class by IBM
Link: https://cognitiveclass.ai/courses/docker-essentials
Time: 3-5 days
Cost: Free + Free Badge

2. Kubernetes Basics — Google (Free)

Platform: Google Cloud Skills Boost
Link: https://cloudskillsboost.google/course_templates/2
Time: 1 week
Cost: Free

🟡 Medium Priority (Do These Next)
3. Oracle Java SE Certification

Platform: Oracle University
Link: https://education.oracle.com/java-se-17-developer/pexam_1Z0-829
Time: 4-6 weeks prep
Cost: ~$245 (most valued Java cert globally)

4. AWS Cloud Practitioner

Platform: AWS Training
Link: https://aws.amazon.com/certification/certified-cloud-practitioner
Time: 2-4 weeks prep
Cost: ~$100 exam / Free study material

🟢 Quick Wins (Easiest to Get)
5. Spring Framework — Udemy

Search: "Spring Boot 3 Udemy Chad Darby"
Link: https://www.udemy.com/course/spring-hibernate-tutorial
Time: 1-2 weeks
Cost: ~₹499 during sale (goes on sale very frequently)

6. Kafka for Developers — Udemy

Since Kafka is already on your CV, getting certified validates it
Search: "Apache Kafka Series Udemy Stephane Maarek"
Link: https://www.udemy.com/course/apache-kafka
Time: 1-2 weeks
Cost: ~₹499 during sale

📋 Recommended Order
Week 1-2:  Docker Essentials (Free — quick win)
Week 3-4:  Kubernetes Basics (Free — quick win)
Month 2:   Spring Boot or Kafka on Udemy (cheap)
Month 3-4: AWS Cloud Practitioner (high value)
Month 5-6: Oracle Java SE (highest value, needs most prep)

Build One Java Project (Most Important — 2-3 days)
This is the biggest gap. Here's the simplest project that looks great:
Project: Library Management REST API

What it does:
- Add, update, delete, get books
- Simple Spring Boot + REST API
- H2 in-memory database (no setup needed)
- Swagger UI for documentation

Folder structure to follow: 
library-management-api/
├── src/
│   ├── controller/
│   │   └── BookController.java
│   ├── service/
│   │   └── BookService.java
│   ├── repository/
│   │   └── BookRepository.java
│   └── model/
│       └── Book.java
├── README.md
└── pom.xml

ReadMe Template for this project:
## 📚 Library Management REST API

A RESTful API built with Java and Spring Boot 
for managing a library's book inventory.

### 🛠️ Tech Stack
Java | Spring Boot | REST API | H2 Database | Swagger

### 🚀 Features
- CRUD operations for books
- Search by author/title
- Swagger UI documentation
- Unit tests with JUnit

### ▶️ How to Run
git clone https://github.com/tshambhavi/library-management-api
cd library-management-api
mvn spring-boot:run

### 📌 API Endpoints
GET    /api/books       - Get all books
GET    /api/books/{id}  - Get book by ID
POST   /api/books       - Add new book
PUT    /api/books/{id}  - Update book
DELETE /api/books/{id}  - Delete book