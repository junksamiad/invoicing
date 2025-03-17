
# Invoice System Architecture

```mermaid
graph TD
    subgraph Frontend
        UI[Web Interface]
        Form[Invoice Form]
        Preview[PDF Preview]
    end

    subgraph Backend
        API[FastAPI Server]
        PDFGen[PDF Generator]
        Valid[Validation Layer]
    end

    subgraph Database
        PG[(PostgreSQL)]
        CompanyTB[Company Details]
        InvoiceTB[Invoices]
    end

    subgraph Storage
        FS[File Storage]
    end

    UI --> Form
    Form --> API
    UI --> Preview
    
    API --> Valid
    Valid --> PDFGen
    Valid --> PG
    
    PDFGen --> FS
    
    PG --> CompanyTB
    PG --> InvoiceTB
    
    API --> UI
    FS --> Preview
```

## Data Flow
1. User inputs data through web interface
2. Data validated by backend
3. Invoice generated and stored as PDF
4. Invoice details saved to database
5. PDF path stored in database
6. Invoice accessible for viewing/downloading
