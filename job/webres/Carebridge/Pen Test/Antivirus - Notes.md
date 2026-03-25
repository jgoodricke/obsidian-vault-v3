## Architecture Requirements

### Abstractions

Introduce the following interfaces/services:

- `DocumentScanOrchestrator`
    
- `DocumentScanner`
    
- `ScanResultHandler`
    
- `DocumentPromotionService`
    
- `TemporaryFileStore`
    

### Implementation

- Use **SynchronousDocumentScanOrchestrator** for pilot
    
- Bind via service container to allow swapping later
    
- Ensure scan logic is **not embedded in controller**
    

### Domain Model

`UploadedDocument` must include:

- status (`pending`, `clean`, `infected`, `failed`)
    
- temporary_path
    
- final_path
    
- scanned_at
    
- failure_reason
    

---

## Frontend / API Considerations

- API must return document status:
    
    - `pending`
        
    - `clean`
        
    - `infected`
        
    - `failed`
        
- UI should support `pending` state even if rarely used in pilot
    
- Do not assume upload = immediately usable

## Notes

- This is a **pilot implementation** prioritising simplicity
    
- Future upgrade path:
    
    - Replace orchestrator with queue-based or S3 event-driven scanning
        
- Keep design aligned with eventual national rollout, but avoid premature complexity
    

---