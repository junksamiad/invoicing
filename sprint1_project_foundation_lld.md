
# Low-Level Design: Project Foundation
## Sprint 1 Technical Specification

### Overview
This sprint focuses on establishing the foundational architecture for the invoicing system. We're setting up a robust FastAPI application with proper separation of concerns, database integration, and initial company management features. The foundation sprint is critical as it sets up the architectural patterns and best practices that will be followed throughout the project lifecycle.

### Implementation Steps

1. **Project Structure Setup**
   The project structure follows a modular design pattern to ensure scalability and maintainability. Each component has a specific responsibility and is organized in a way that makes the codebase easy to navigate and understand.
   - Creating a modular structure separating routes, models, and business logic
   - Implementing a clean architecture pattern for maintainability
   - Setting up configuration management for different environments

2. **Database Integration**
   Database integration is implemented using SQLAlchemy ORM to provide a robust and type-safe interface to our PostgreSQL database. This layer handles all data persistence operations and ensures data integrity.
   - Establishing PostgreSQL connection using SQLAlchemy ORM
   - Creating initial models for company data
   - Setting up migration system for schema changes
   - Implementing session management and connection pooling

3. **API Layer**
   The API layer serves as the interface between the frontend and our business logic. It's built using FastAPI to leverage automatic OpenAPI documentation, type checking, and high performance.
   - Building RESTful endpoints for company management
   - Implementing input validation using Pydantic models
   - Setting up error handling and response formatting
   - Adding basic authentication structure

4. **Template System**
   The template system provides server-side rendering capabilities using Jinja2. It's structured to support component reuse and maintain consistent styling across the application.
   - Creating base templates for consistent UI
   - Setting up static file serving
   - Implementing basic layout structure

### Key Technical Decisions

Each technical choice has been made with specific benefits in mind:

- **FastAPI Framework**: Selected for its modern features, async support, and automatic OpenAPI documentation. FastAPI provides excellent performance metrics and built-in support for type checking through Pydantic.

- **SQLAlchemy ORM**: Chosen for its robust type safety and powerful querying capabilities. It provides a pythonic interface to database operations while maintaining flexibility for complex queries.

- **Pydantic**: Implements runtime data validation and serialization. This ensures that all data flowing through the application meets our defined schemas and types.

- **Jinja2**: Our templating engine choice for server-side rendering, providing powerful template inheritance and macro capabilities.

### 1. Project Structure
The project structure follows a domain-driven design approach, separating concerns into distinct layers:

```
/invoicing
├── app/                      # Main application package
│   ├── __init__.py          # Package initializer
│   ├── main.py              # Application entry point
│   ├── config.py            # Configuration management
│   ├── api/                 # API layer
│   │   ├── __init__.py     
│   │   ├── endpoints/       # Route handlers
│   │   │   ├── company.py   # Company management endpoints
│   │   │   └── invoice.py   # Invoice management endpoints
│   ├── core/                # Core functionality
│   │   ├── security.py      # Authentication & authorization
│   │   └── config.py        # Core configuration
│   ├── db/                  # Database layer
│   │   ├── session.py       # Database session management
│   │   └── base.py         # Base model class
│   ├── models/              # SQLAlchemy models
│   │   ├── company.py       # Company data model
│   │   └── invoice.py       # Invoice data model
│   └── schemas/             # Pydantic models
│       ├── company.py       # Company data validation
│       └── invoice.py       # Invoice data validation
└── templates/               # Jinja2 templates
    └── base.html           # Base template layout
```

### 2. Database Implementation
The database layer is designed for reliability and performance:

#### Connection Setup (db/session.py)
Implements connection pooling and session management:

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "postgresql://user:password@0.0.0.0/invoicing"
engine = create_engine(SQLALCHEMY_DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
```

#### Base Models (models/company.py)
Implements the company data model with audit trails:

```python
from sqlalchemy import Column, Integer, String, DateTime
from sqlalchemy.sql import func
from app.db.base import Base

class Company(Base):
    __tablename__ = "company_details"
    
    id = Column(Integer, primary_key=True, index=True)
    company_name = Column(String(255), nullable=False)
    address_line1 = Column(String(255), nullable=False)
    address_line2 = Column(String(255))
    city = Column(String(100))
    postal_code = Column(String(20))
    country = Column(String(100))
    vat_number = Column(String(50))
    company_number = Column(String(50))
    created_at = Column(DateTime(timezone=True), server_default=func.now())
    updated_at = Column(DateTime(timezone=True), onupdate=func.now())
```

### 3. API Endpoints
The API layer implements RESTful principles and proper error handling:

#### Company Management (api/endpoints/company.py)
```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from app.schemas.company import CompanyCreate, CompanyUpdate
from app.models.company import Company

router = APIRouter()

@router.post("/companies/")
def create_company(company: CompanyCreate, db: Session = Depends(get_db)):
    db_company = Company(**company.dict())
    db.add(db_company)
    db.commit()
    db.refresh(db_company)
    return db_company

@router.get("/companies/{company_id}")
def read_company(company_id: int, db: Session = Depends(get_db)):
    company = db.query(Company).filter(Company.id == company_id).first()
    if company is None:
        raise HTTPException(status_code=404, detail="Company not found")
    return company
```

### 4. Template System
The template system provides a consistent look and feel:

#### Base Template (templates/base.html)
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}{% endblock %} - Adaptix Innovation Invoicing</title>
    <link rel="stylesheet" href="{{ url_for('static', path='/styles.css') }}">
</head>
<body>
    <nav>
        <!-- Navigation components -->
    </nav>
    <main>
        {% block content %}{% endblock %}
    </main>
    <footer>
        <!-- Footer components -->
    </footer>
</body>
</html>
```

### 5. Testing Strategy
Comprehensive testing approach covering multiple layers:
- Unit tests for database models ensuring data integrity
- Integration tests for API endpoints verifying business logic
- Template rendering tests checking UI consistency
- Database migration tests validating schema changes

### 6. Dependencies
Core dependencies required for the project:
```toml
[dependencies]
fastapi
uvicorn
sqlalchemy
psycopg2-binary
pydantic
jinja2
python-multipart
```

### 7. Configuration Management
Environment configuration handling:
#### Environment Variables
- `DATABASE_URL`: Database connection string
- `SECRET_KEY`: Application security key
- `DEBUG_MODE`: Development mode flag
- `API_PREFIX`: API routing prefix

### 8. Security Considerations
Multiple layers of security implementation:
- Input validation using Pydantic models
- SQL injection prevention through SQLAlchemy
- CORS configuration for API security
- Rate limiting implementation to prevent abuse
