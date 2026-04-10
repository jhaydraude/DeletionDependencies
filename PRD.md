# PRD: Retriever Deletion Dependencies

**Feature Name:** Retriever Deletion: Show Dependencies with Navigable Links
**Product Area:** Data Cloud - AI Models - Retrievers
**Document Version:** 1.0
**Last Updated:** April 10, 2026

**Key Contacts:**
- **Product Management:** Jeremy Hay Draude
- **Engineering:** Ruiting Zhai
- **UX Design:** Som Liengtiraphan

**Related Documents:**
- [Technical Spike Document](https://salesforce.quip.com/O9bMAD7DNw0W)

---

## Executive Summary

This feature enhances the retriever deletion experience by providing users with a clear, actionable view of all dependencies that block deletion. Instead of a generic error message requiring manual detective work, users will see a comprehensive list of dependencies directly in the error modal, with clickable navigation links that take them directly to each dependent component for removal.

---

## Problem Statement

### Current State
When users attempt to delete or deactivate a retriever that has dependencies, they receive a generic error message:
> "To activate, deactivate, or delete this retriever, remove or deactivate all references to it and try again."

This message provides no visibility into:
- What components reference the retriever
- How many dependencies exist
- Where to find these dependencies
- How to navigate to them

Users must manually search across multiple areas (Agent Actions, Flows, Prompt Templates, Ensemble Retrievers, Data Libraries) to identify and remove references - a time-consuming and frustrating process.

### Impact
- **Poor User Experience:** Users are left guessing where dependencies might exist

---

## Goals & Success Criteria

### Primary Goals
1. **Visibility:** Show users all dependencies that block retriever deletion
2. **Actionability:** Provide direct navigation to each dependent component
3. **Efficiency:** Reduce time to identify and remove dependencies by 80%
4. **Consistency:** Follow existing Salesforce patterns for dependency management (e.g., DMO deletion flow)

### Success Metrics
- TBD

---

## User Personas

### Primary: Salesforce Admin
- **Role:** Manages Data Cloud configurations
- **Need:** Clean up unused or test retrievers
- **Pain Point:** Cannot easily identify what's blocking deletion

### Secondary: AI/ML Specialist
- **Role:** Designs and maintains AI retriever configurations
- **Need:** Refactor retriever architecture
- **Pain Point:** Must manually audit all potential dependency locations

---

## User Stories

### Epic: Improved Retriever Deletion Experience

**Story 1: View Dependencies in Error Modal**
```
As a Data Cloud admin
When I attempt to delete a retriever that has dependencies
I want to see all blocking dependencies directly in the error modal
So that I can immediately understand what needs to be removed
```

**Acceptance Criteria:**
- Error modal displays after delete confirmation when dependencies exist
- All dependency types are shown (Agent Actions, Flows, Prompt Templates, Ensemble Retrievers, Data Libraries, Companion Org References)
- Dependencies are organized by type with count indicators
- Modal is scrollable if dependency list is long

**Story 2: Navigate to Dependencies**
```
As a Data Cloud admin
When viewing dependencies in the error modal
I want to click on each dependency to navigate directly to its setup page
So that I can quickly remove the reference
```

**Acceptance Criteria:**
- Each local dependency shows as a clickable link
- Clicking a dependency opens the appropriate setup page in a new context
- API names are displayed for identification
- Navigation works for all dependency types

**Story 3: View Dependencies in Integrations Tab**
```
As a Data Cloud admin
When I want to see all retriever dependencies without attempting deletion
I want to view them in the Integrations tab
So that I can plan refactoring or understand retriever usage
```

**Acceptance Criteria:**
- Integrations tab exists on retriever detail page
- Dependencies are organized by type (collapsible groups)
- All information from error modal is available in the tab
- Tab is accessible at any time, not just during deletion

**Story 4: Understand Companion Org References**
```
As a Data Cloud admin in a multi-org environment
When viewing dependencies
I want to see which references come from companion orgs
So that I understand which dependencies I can control
```

**Acceptance Criteria:**
- Companion org references are visually distinguished
- Org name is displayed (e.g., "Sales Cloud LOB")
- These references are not clickable (cross-org navigation not supported)
- Clear indication that these are informational only

---

## Functional Requirements

### FR-1: API Endpoint
**Requirement:** Create new REST API endpoint for fetching retriever dependencies

**Specification:**
- **Endpoint:** `GET /ssot/ml/retrievers/{id}/integrations`
- **Response Time:** < 500ms
- **Response Format:**
```json
{
  "references": [
    {
      "id": "string",
      "name": "string",
      "label": "string",
      "type": "Agent|Flow|PromptTemplate|EnsembleRetriever|DataLibrary|Other",
      "navigationUrl": "string|null",
      "isRemoteReference": boolean,
      "orgOwner": "string|null"
    }
  ]
}
```

**Implementation Notes:**
- Reuses existing `MlRetrieverDependencyHelper.fetchDependencies()`
- Constructs navigation URLs server-side
- Enriches companion org references with org display names
- Separate endpoint to avoid latency impact on main retriever GET

### FR-2: Dependency Discovery
**Requirement:** Identify all components that reference a retriever

**Discovery Strategies:**
1. **Agent Actions:** GenAiFunctionDefinition FK
2. **Data Libraries:** AiGroundingSource FK
3. **Ensemble Retrievers:** AIRtrvrDefVerEsmbRtrvr FK
4. **Flows:** InteractionActionCall tracing
5. **Prompt Templates:** GenAiPromptTemplateVersion tracing
6. **Generic:** EntityInfo-based FK discovery
7. **Cross-Org:** CReferences.whatReferencesThis()

### FR-3: Navigation URL Construction
**Requirement:** Build Lightning setup page URLs for each dependency type

**URL Patterns:**
| Type | URL Template | Deep Link |
|------|--------------|-----------|
| Agent Action | `/lightning/setup/AgentAssetLibrary/{id}/editAction` | Yes |
| Flow | `/lightning/setup/Flows/page?address=/{id}` | Yes |
| Prompt Template | `/lightning/setup/PromptTemplateSetup/{id}/view` | Yes |
| Ensemble Retriever | `/lightning/n/standard-EinsteinStudio?c__assetId={id}` | Yes |
| Data Library | `/lightning/setup/EinsteinDataLibrary/home` | List page only |
| Companion Org | `null` | No (cross-org) |
| Other | `null` | May not work |

### FR-3a: Dependency Display Scenarios
**Requirement:** Present dependencies with appropriate information and navigation based on availability

Dependencies are displayed according to three distinct scenarios based on what information and navigation capabilities are available:

#### Scenario 1: Known Entities with UI Navigation
**Applies to:** Agent Actions, Flows, Prompt Templates, Ensemble Retrievers, Data Libraries

**Display Elements:**
- Entity Label (user-friendly name)
- Entity Name (API/developer name)
- Record ID
- Clickable navigation link to setup page

**Example:**
```
My Retriever Action
API Name: MyRetrieverAction
[Clickable blue link → opens /lightning/setup/AgentAssetLibrary/{id}/editAction]
```

**Behavior:** Users can click the entity name to navigate directly to the component's setup page for editing or removal.

---

#### Scenario 2: Local Dependencies Without UI Pages
**Applies to:** Other teams' entities with foreign key references, unclassified entity types, entities without setup pages

**Display Elements:**
- Entity Name (API/developer name or type)
- Record ID (required for identification)
- API Name (if available)
- Text indicator: "No UI page available"

**Examples:**
```
SearchAnswersIndexConfig
Record ID: 0DI5g000000GmKlGAK • No UI page available
[Plain text, not clickable]

AIRetrieverFieldMapping
Record ID: 1CzS-B0000007cBt0AI • API Name: Basic_KA_LOB_1Cy_ASPb98816a1
[Plain text, not clickable]
```

**Behavior:** Displayed as plain text (not clickable). Users must use the Record ID to locate and manage these dependencies through alternative means (e.g., API, Workbench, or contacting the owning team).

**Why no link:** These entities either:
- Have foreign key relationships to retrievers but no dedicated UI
- Belong to other teams that haven't provided setup pages
- Are discovered generically and not yet classified
- Are internal/system entities without user-facing interfaces

---

#### Scenario 3: Companion Org Remote References
**Applies to:** Dependencies from companion orgs in multi-org Data Cloud environments

**Display Elements:**
- Entity Name (API/developer name)
- Record ID
- API Name (if available)
- Companion org badge displaying org name (e.g., "Sales Cloud LOB")

**Example:**
```
RemoteAgentAction
Record ID: 172YY00000Qrst
[Sales Cloud LOB badge]
[Plain text, not clickable]
```

**Behavior:** Displayed as plain text (not clickable) with visual badge indicating the companion org. These are informational only - users cannot navigate from the home org to the companion org's UI.

**Why no link:**
- Retriever is synced from home org to companion org
- Used by flows, prompt templates, or other components in the companion org
- No tool exists to construct cross-org navigation URLs
- Different org context with separate authentication and UI

**User Action Required:** Contact the companion org administrator or use companion org-specific tools to manage these references.

---

**Summary Table:**

| Scenario | Link Available | Display Style | Information Shown | User Action |
|----------|---------------|---------------|-------------------|-------------|
| Known Entities | ✅ Yes | Clickable blue link | Label, API Name, Record ID | Click to navigate and edit |
| Local No-UI | ❌ No | Plain text | Entity Name, Record ID, optional API Name | Use Record ID via API/Workbench |
| Companion Org | ❌ No | Plain text + badge | Entity Name, Record ID, Org badge | Contact companion org admin |

### FR-4: Error Modal UI
**Requirement:** Display improved error modal with dependency list

**Components:**
- **Header:** Red background, "You can't delete this retriever"
- **Body:**
  - Explanation text
  - Collapsible dependency groups by type
  - Clickable dependency names (blue links)
  - API names displayed below each label
  - Companion org badges
- **Footer:** "Cancel" button
- **Size:** 800px wide, scrollable if needed

### FR-5: Integrations Tab
**Requirement:** Add Integrations tab to retriever detail page

**Components:**
- Tab appears next to "Overview" tab
- Shows same dependency structure as error modal
- Includes explanatory text
- Available at all times (not just during deletion)

### FR-6: Delete Confirmation Flow
**Requirement:** Update delete confirmation to check dependencies

**Flow:**
1. User clicks "Delete Retriever"
2. Show warning confirmation modal
3. User clicks "Delete" in confirmation
4. **Check for dependencies:**
   - If no dependencies → Delete succeeds
   - If dependencies exist → Show error modal with dependency list
5. User can click links to navigate to dependencies
6. User removes dependencies
7. User retries deletion → Success

---

## Non-Functional Requirements

### NFR-1: Performance
- Dependency check: < 500ms
- Modal rendering: < 100ms
- Navigation: < 200ms to target page

### NFR-2: Scalability
- Support up to 100 dependencies per retriever
- Graceful handling of large dependency lists (scrolling, pagination if needed)

### NFR-3: Reliability
- 99.9% uptime for dependency endpoint
- Graceful degradation if dependency check fails (show generic error)

### NFR-4: Security
- Respect user permissions for navigation URLs
- Only show dependencies user has permission to view
- No sensitive information in error messages

### NFR-5: Accessibility
- WCAG 2.1 AA compliant
- Keyboard navigation support
- Screen reader compatible

### NFR-6: Browser Support
- All Salesforce supported browsers
- Responsive design for different screen sizes

---

## UI/UX Requirements

### UX-1: Error Modal Design
- **Visual Hierarchy:** Red header for error severity
- **Scannability:** Group dependencies by type
- **Affordance:** Blue links indicate clickability
- **Clarity:** Non-clickable items use plain text
- **Context:** Show API names for technical users

### UX-2: Integrations Tab
- **Discoverability:** Tab visible at all times
- **Consistency:** Same structure as error modal
- **Guidance:** Explanatory text at top

### UX-3: Dependency Groups
- **Organization:** Collapsible sections by type
- **Count Indicators:** Show number of dependencies per type
- **Default State:** Collapsed to reduce cognitive load
- **Expansion:** Click header to expand/collapse

### UX-4: Companion Org References
- **Visual Distinction:** Badge with org name
- **Non-Interactivity:** Plain text (not link)
- **Tooltip:** Consider adding explanation on hover

### UX-5: Loading States
- **Dependency Check:** Show spinner during check
- **Navigation:** Indicate when link is clicked

---

## Technical Requirements

### Tech-1: Backend Implementation

**Files to Modify:**
1. `MlRetrieverReferenceTypeEnum.java` - Add Agent, DataLibrary, Other enums
2. `RetrieverReferenceRepresentation.java` - Add navigationUrl, isRemoteReference, orgOwner fields
3. `MlRetrieverReferenceRepresentation.java` - Add @ConnectOutputProperty annotations
4. `MlRetrieverRepresentationBuilderExt.java` - Wire new fields into builder
5. `RepresentationBuilderFactory.java` - Add builder method
6. `CdpMlConnectFeatureImpl.java` - Register new resource
7. `MlRetrieverService.java` - Add getRetrieverIntegrations() method
8. `MlRetrieverServiceImpl.java` - Implement getRetrieverIntegrations(), buildNavigationUrl()
9. `AIRetrieverDefinitionObject.java` - Update delete error message

**New Files:**
1. `MlRetrieverIntegrationsResource.java` - ConnectAPI resource interface
2. `MlRetrieverIntegrationsRepresentation.java` - Output representation
3. `MlRetrieverIntegrationsRepresentationBuilderExt.java` - Builder extension
4. `MlRetrieverIntegrationsResourceImpl.java` - Resource implementation

### Tech-2: Frontend Implementation

**Components:**
1. Retriever detail page - Add Integrations tab
2. Delete flow - Update to check dependencies
3. Error modal - Display dependency list
4. Dependency list component - Reusable for tab and modal

**Framework:** Lightning Web Components (LWC) or Aura

### Tech-3: Data Model

**No schema changes required** - All data exists, only exposing via new endpoint

### Tech-4: Testing Requirements

**Unit Tests:**
- MlRetrieverServiceImpl.getRetrieverIntegrations()
- Navigation URL construction for each type
- Companion org enrichment logic

**Integration Tests:**
- End-to-end dependency check
- API endpoint response format
- Error scenarios (no dependencies, API failure)

**UI Tests:**
- Modal display on delete with dependencies
- Link navigation
- Tab display
- Collapsible groups

**Manual Testing Scenarios:**
1. Delete retriever with no dependencies → Success
2. Delete retriever with each dependency type → Error modal shows correctly
3. Click each dependency type link → Navigates correctly
4. View Integrations tab → All dependencies shown
5. Companion org reference → Badge shown, not clickable
6. Large number of dependencies → Modal scrolls

---

## Dependencies & Integrations

### Internal Dependencies
- `MlRetrieverDependencyHelper` - Existing dependency discovery logic
- `RemoteReferenceService` - Companion org reference data
- `RemoteConnectionSetupService` - Org name resolution
- `EntityInfo` - Entity type classification
- Lightning Design System - UI components

### External Dependencies
- None

---

## Rollout Plan

### Phase 1: Backend Implementation (2 weeks)
- Implement API endpoint
- Add navigation URL construction
- Enrich companion org references
- Unit tests

### Phase 2: Frontend Implementation (2 weeks)
- Build Integrations tab
- Update delete flow
- Create error modal with dependencies
- UI tests

### Phase 3: Integration Testing (1 week)
- End-to-end testing
- Performance testing
- Accessibility review
- Security review

### Phase 4: Beta Release (2 weeks)
- Release to internal users
- Gather feedback
- Monitor metrics
- Fix bugs

### Phase 5: GA Release (1 week)
- Release to all users
- Monitor support tickets
- Update documentation
- Track success metrics

---

## Open Questions & Future Considerations

### Open Questions
1. **Navigation Behavior:** Should links open in new tab/window or same context?
2. **Data Library Deep Links:** Can we create direct links to specific libraries?
3. **Permission Handling:** How to handle dependencies user can't access?
4. **Bulk Operations:** Should we support bulk dependency resolution?

### Future Enhancements
1. **Bulk Actions:** "Remove All References" button for certain dependency types
2. **Dependency Graph:** Visual representation of retriever dependencies
3. **Impact Analysis:** Show what would be affected by deletion
4. **Dependency Filtering:** Search/filter in Integrations tab
5. **Export:** Download dependency list as CSV
6. **Notifications:** Alert users when dependencies are added/removed
7. **Automated Cleanup:** Suggest or auto-remove stale dependencies

### Out of Scope
- Cascade delete (automatically delete all dependencies)
- Cross-org dependency resolution
- Undo delete functionality
- Version-specific dependency tracking

---




---

## Appendix

### Appendix A: Mockup
See `retriever-deletion-mockup.html` for interactive prototype

### Appendix B: API Response Example
```json
{
  "references": [
    {
      "id": "172SB00000FQrxhYAD",
      "name": "MyRetrieverAction",
      "label": "My Retriever Action",
      "type": "Agent",
      "navigationUrl": "/lightning/setup/AgentAssetLibrary/172SB00000FQrxhYAD/editAction",
      "isRemoteReference": false,
      "orgOwner": null
    },
    {
      "id": "172YY00000Qrst",
      "name": "RemoteAgentAction",
      "label": "RemoteAgentAction",
      "type": "Agent",
      "navigationUrl": null,
      "isRemoteReference": true,
      "orgOwner": "Sales Cloud LOB"
    }
  ]
}
```

### Appendix C: Dependency Type Matrix

| Dependency Type | Blocks Deletion? | Clickable Link? | API Name Shown? | Special Display |
|-----------------|------------------|-----------------|-----------------|-----------------|
| Agent Action | Yes | Yes | Yes | - |
| Flow | Yes | Yes | Yes | - |
| Prompt Template | Yes | Yes | Yes | - |
| Ensemble Retriever | Yes | Yes | Yes | - |
| Data Library | Yes | Yes (list page) | Yes | - |
| Companion Org | Yes* | No | Yes | Org badge |
| Other | Maybe | Maybe | Yes | - |

*Companion org refs block deletion in local org but cannot be directly resolved

---

## References

### Technical Documentation
- [Technical Spike: Retriever Deletion Dependencies](https://salesforce.quip.com/O9bMAD7DNw0W) - Detailed technical implementation approach, API specifications, and backend architecture

### Design Assets
- `retriever-deletion-mockup.html` - Interactive prototype demonstrating the feature
- `CLAUDE.md` - Technical implementation guide for developers

---

**Document Approval:**

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Product Manager | Jeremy Hay Draude | | |
| Engineering Lead | Ruiting Zhai | | |
| UX Designer | Som Liengtiraphan | | |
| Tech Lead | | | | |
