
# Low-Level Design: Project Foundation
## Sprint 1 Technical Specification

### 1. Project Structure
```
/invoicing
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── config.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── endpoints/
│   │   │   ├── company.py
│   │   │   └── invoice.py
│   ├── core/
│   │   ├── security.py
│   │   └── config.py
│   ├── db/
│   │   ├── session.py
│   │   └── base.py
│   ├── models/
│   │   ├── company.py
│   │   └── invoice.py
│   └── schemas/
│       ├── company.py
│       └── invoice.py
└── templates/
    └── base.html
```

### 2. Database Implementation
#### Connection Setup (db/session.py)
```python
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "postgresql://user:password@localhost/invoicing"
engine = create_engine(SQLALCHEMY_DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
```

#### Base Models (models/company.py)
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
- Unit tests for database models
- Integration tests for API endpoints
- Template rendering tests
- Database migration tests

### 6. Dependencies
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
#### Environment Variables
- `DATABASE_URL`
- `SECRET_KEY`
- `DEBUG_MODE`
- `API_PREFIX`

### 8. Security Considerations
- Input validation using Pydantic models
- SQL injection prevention through SQLAlchemy
- CORS configuration
- Rate limiting implementation
