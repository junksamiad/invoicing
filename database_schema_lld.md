
# Database Schema Low-Level Design
## Adaptix Innovation Invoicing System

### Overview
This document details the database schema design for the invoicing system, including table structures, relationships, and constraints.

### Tables

#### 1. company_details
Stores business information for companies using the system.

| Column         | Type         | Constraints           | Description                               |
|---------------|--------------|----------------------|-------------------------------------------|
| id            | SERIAL       | PRIMARY KEY          | Unique identifier for the company         |
| company_name  | VARCHAR(255) | NOT NULL            | Legal name of the company                 |
| address_line1 | VARCHAR(255) | NOT NULL            | Primary address line                      |
| address_line2 | VARCHAR(255) |                     | Secondary address line (optional)         |
| city          | VARCHAR(100) | NOT NULL            | City location                             |
| postal_code   | VARCHAR(20)  | NOT NULL            | Postal/ZIP code                           |
| country       | VARCHAR(100) | NOT NULL            | Country name                              |
| vat_number    | VARCHAR(50)  |                     | VAT registration number                   |
| company_number| VARCHAR(50)  |                     | Company registration number               |
| created_at    | TIMESTAMP    | DEFAULT NOW()       | Record creation timestamp                 |
| updated_at    | TIMESTAMP    | DEFAULT NOW()       | Last update timestamp                     |

#### 2. invoices
Stores invoice records and their details.

| Column         | Type          | Constraints                    | Description                               |
|---------------|---------------|--------------------------------|-------------------------------------------|
| id            | SERIAL        | PRIMARY KEY                    | Unique identifier for the invoice         |
| invoice_number| VARCHAR(50)   | UNIQUE, NOT NULL               | Human-readable invoice identifier         |
| company_id    | INTEGER       | FOREIGN KEY(company_details.id)| Reference to issuing company             |
| client_name   | VARCHAR(255)  | NOT NULL                       | Name of the client                       |
| client_email  | VARCHAR(255)  | NOT NULL                       | Client's email address                   |
| client_address| TEXT          | NOT NULL                       | Client's full address                    |
| issue_date    | DATE          | NOT NULL, DEFAULT CURRENT_DATE | Date invoice was issued                  |
| due_date      | DATE          | NOT NULL                       | Payment due date                         |
| items         | JSONB         | NOT NULL                       | Array of line items                      |
| subtotal      | DECIMAL(10,2) | NOT NULL                       | Sum before tax                           |
| tax_rate      | DECIMAL(5,2)  | NOT NULL, DEFAULT 20.0        | Applied tax rate percentage              |
| tax_amount    | DECIMAL(10,2) | NOT NULL                       | Calculated tax amount                    |
| total         | DECIMAL(10,2) | NOT NULL                       | Final total including tax                |
| status        | VARCHAR(50)   | DEFAULT 'draft'                | Invoice status (draft/sent/paid)         |
| created_at    | TIMESTAMP     | DEFAULT NOW()                  | Record creation timestamp                |
| updated_at    | TIMESTAMP     | DEFAULT NOW()                  | Last update timestamp                    |

### Indexes

```sql
-- company_details indexes
CREATE INDEX idx_company_name ON company_details(company_name);
CREATE INDEX idx_vat_number ON company_details(vat_number);

-- invoices indexes
CREATE INDEX idx_invoice_number ON invoices(invoice_number);
CREATE INDEX idx_company_id ON invoices(company_id);
CREATE INDEX idx_client_email ON invoices(client_email);
CREATE INDEX idx_status ON invoices(status);
CREATE INDEX idx_issue_date ON invoices(issue_date);
```

### Relationships
- One-to-Many relationship between `company_details` and `invoices`
- Each invoice must belong to exactly one company
- A company can have multiple invoices

### Data Types
- Use of JSONB for flexible line items storage
- DECIMAL for precise monetary calculations
- VARCHAR with appropriate length limits
- TEXT for unlimited length strings
- TIMESTAMP for audit trails

### Constraints
- NOT NULL where data is required
- UNIQUE for business identifiers
- Foreign key constraints for referential integrity
- Default values for audit timestamps
- Check constraints for valid monetary amounts

### Security Considerations
- Encrypted storage for sensitive data
- Role-based access control
- Audit trail for all modifications
- Input validation before storage
