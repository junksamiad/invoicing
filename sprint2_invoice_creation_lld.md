
# Low-Level Design: Invoice Creation
## Sprint 2 Technical Specification

### 1. Frontend Components
#### Invoice Form Structure
```html
<!-- templates/invoice/create.html -->
{% extends "base.html" %}
{% block content %}
<form id="invoiceForm" class="invoice-form">
    <!-- Client Information -->
    <section class="client-info">
        <input type="text" name="client_name" required>
        <input type="email" name="client_email" required>
        <textarea name="client_address" required></textarea>
    </section>

    <!-- Invoice Items -->
    <section class="invoice-items">
        <div class="item-list" id="itemList">
            <!-- Dynamic item rows -->
        </div>
        <button type="button" id="addItem">Add Item</button>
    </section>

    <!-- Totals -->
    <section class="totals">
        <div id="subtotal"></div>
        <div id="tax"></div>
        <div id="total"></div>
    </section>
</form>
{% endblock %}
```

### 2. Database Models
#### Invoice Model (models/invoice.py)
```python
from sqlalchemy import Column, Integer, String, Date, Decimal, ForeignKey, JSON
from sqlalchemy.sql import func
from app.db.base import Base

class Invoice(Base):
    __tablename__ = "invoices"
    
    id = Column(Integer, primary_key=True, index=True)
    invoice_number = Column(String(50), unique=True)
    company_id = Column(Integer, ForeignKey("company_details.id"))
    client_name = Column(String(255), nullable=False)
    client_email = Column(String(255), nullable=False)
    client_address = Column(String, nullable=False)
    issue_date = Column(Date, default=func.current_date())
    due_date = Column(Date, nullable=False)
    items = Column(JSON, nullable=False)
    subtotal = Column(Decimal(10,2), nullable=False)
    tax_rate = Column(Decimal(5,2), default=20.0)
    tax_amount = Column(Decimal(10,2), nullable=False)
    total = Column(Decimal(10,2), nullable=False)
    status = Column(String(50), default="draft")
```

### 3. API Endpoints
#### Invoice Management (api/endpoints/invoice.py)
```python
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session
from app.schemas.invoice import InvoiceCreate, InvoiceResponse
from app.models.invoice import Invoice
from app.utils.invoice_number import generate_invoice_number

router = APIRouter()

@router.post("/invoices/", response_model=InvoiceResponse)
def create_invoice(invoice: InvoiceCreate, db: Session = Depends(get_db)):
    invoice_data = invoice.dict()
    invoice_data["invoice_number"] = generate_invoice_number()
    
    # Calculate totals
    items = invoice_data["items"]
    subtotal = sum(item["quantity"] * item["price"] for item in items)
    tax_amount = subtotal * (invoice_data["tax_rate"] / 100)
    total = subtotal + tax_amount
    
    invoice_data.update({
        "subtotal": subtotal,
        "tax_amount": tax_amount,
        "total": total
    })
    
    db_invoice = Invoice(**invoice_data)
    db.add(db_invoice)
    db.commit()
    db.refresh(db_invoice)
    return db_invoice

@router.get("/invoices/", response_model=list[InvoiceResponse])
def list_invoices(
    skip: int = 0,
    limit: int = 10,
    db: Session = Depends(get_db)
):
    return db.query(Invoice).offset(skip).limit(limit).all()
```

### 4. Frontend Logic
#### Invoice Form Handler (static/js/invoice-form.js)
```javascript
class InvoiceForm {
    constructor() {
        this.items = [];
        this.taxRate = 20.0;
        this.setupEventListeners();
    }

    setupEventListeners() {
        document.getElementById('addItem').addEventListener('click', 
            () => this.addItemRow());
        
        document.getElementById('invoiceForm').addEventListener('submit',
            (e) => this.handleSubmit(e));
    }

    calculateTotals() {
        const subtotal = this.items.reduce((sum, item) => 
            sum + (item.quantity * item.price), 0);
        const taxAmount = subtotal * (this.taxRate / 100);
        const total = subtotal + taxAmount;

        return { subtotal, taxAmount, total };
    }

    async handleSubmit(e) {
        e.preventDefault();
        const formData = new FormData(e.target);
        const invoiceData = {
            client_name: formData.get('client_name'),
            client_email: formData.get('client_email'),
            client_address: formData.get('client_address'),
            items: this.items,
            tax_rate: this.taxRate,
            due_date: formData.get('due_date')
        };

        try {
            const response = await fetch('/api/invoices/', {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify(invoiceData)
            });
            if (response.ok) {
                window.location.href = '/invoices/';
            }
        } catch (error) {
            console.error('Error creating invoice:', error);
        }
    }
}
```

### 5. Validation Schemas
#### Invoice Schemas (schemas/invoice.py)
```python
from pydantic import BaseModel, EmailStr, conlist
from decimal import Decimal
from datetime import date
from typing import List

class InvoiceItem(BaseModel):
    description: str
    quantity: int
    price: Decimal
    
class InvoiceCreate(BaseModel):
    client_name: str
    client_email: EmailStr
    client_address: str
    items: conlist(InvoiceItem, min_items=1)
    tax_rate: Decimal = 20.0
    due_date: date
    
class InvoiceResponse(InvoiceCreate):
    id: int
    invoice_number: str
    subtotal: Decimal
    tax_amount: Decimal
    total: Decimal
    status: str
```

### 6. Testing Strategy
- Unit tests for invoice calculations
- API endpoint integration tests
- Form validation tests
- Database operation tests

### 7. Error Handling
- Form validation errors
- Database constraint violations
- Invoice number uniqueness
- Tax calculation edge cases

### 8. Dependencies
- Pydantic for data validation
- SQLAlchemy for database operations
- FastAPI for API endpoints
- JavaScript for frontend interactivity
